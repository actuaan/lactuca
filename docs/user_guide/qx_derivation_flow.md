# qx Derivation Flow

This page traces the complete path from raw mortality data to the final probability values
returned by the public API, covering both integer and fractional starting ages.

## The decrement array: where it all begins

Every life table file (`.ltk`) stores integer-age annual mortality rates $q_x$ — the
probability that a life aged exactly $x$ dies before reaching age $x + 1$.  When a table
object is created, these rates are loaded into memory as a float64 array indexed by age,
one entry per year from age 0 to the terminal age $\omega$.

For **generational tables**, improvement factors are applied at this stage: each base rate
$q_x^{\text{base}}$ is scaled by cohort-specific improvement factors to produce the
cohort-adjusted rate $q_x^{(c)}$ used for all subsequent calculations.  All of this
happens before any rounding.

Once loaded (and possibly modified via {meth}`lactuca.LifeTable.modify_qx`), the
integer-age $q_x$ array is the single source of truth for every probability the table
can return.  Modifications such as `table_combination` (with optional
`combination_mode`) replace or merge this array **before** the $l_x$ recursion; see
{doc}`modifying_decrements`.

:::{note}
Disability and exit tables store $i_x$ (disability inception rates) and $o_x$ (exit rates)
respectively, following the same structure.  The derivation logic described here applies
equally to all three table types.
:::

## Building the survival function

From the integer-age decrement array, a complete survival function $l_x$ is derived via
the classical radix recursion:

$$
l_{x+1} = l_x \cdot (1 - q_x), \qquad l_0 = 1{,}000{,}000
$$

This recursion yields $l_x$ for every integer age $x = 0, 1, \ldots, \omega$, using a
starting cohort of one million lives.  All intermediate multiplications are performed
in full float64 precision — no rounding at this stage.

Two versions of this array are maintained:

- A **high-precision version** keeps full float64 accuracy.  It is used automatically
  when `config.calculation_mode` is set to a continuous mode
  (`"continuous_precision"` or `"continuous_simplified"`), where maximum precision is
  required throughout the calculation pipeline.
- A **rounded version** rounds each $l_x$ entry to `config.decimals.lx` decimal places.
  This is the version used by all discrete calculation modes and by all public methods
  that return $l_x$ values.

```python
from lactuca import LifeTable

lt = LifeTable("PASEM2020_Rel_1o", "m")
print(lt.lx(65))   # returns l_65, rounded to config.decimals.lx
```

## Integer-age annual probability

For an integer age $x$ with the default unit period ($m = 1$), the annual mortality
probability $q_x$ is retrieved directly from the stored integer-age decrement array —
no $l_x$ computation is needed:

$$
q_x = \text{(stored decrement rate at age } x\text{)}
$$

The value is rounded to `config.decimals.qx` on return.  Symmetrically, the annual
survival probability $p_x = 1 - q_x$ is rounded to `config.decimals.px`.

```python
from lactuca import LifeTable

lt = LifeTable("PASEM2020_Rel_1o", "m")
print(lt.qx(65))   # annual probability of death at age 65
print(lt.px(65))   # annual probability of survival at age 65
```

To retrieve the complete array for all ages at once, pass `None`.  The same pattern
applies to `px()`, `lx()`, and `dx()`:

```python
from lactuca import LifeTable

lt = LifeTable("PASEM2020_Rel_1o", "m")
qx_all = lt.qx(None)   # NDArray[float64] of length omega + 1, annual mortality rates
px_all = lt.px(None)   # NDArray[float64] of length omega + 1, annual survival rates
lx_all = lt.lx(None)   # NDArray[float64] of length omega + 1, survivors at each age
dx_all = lt.dx(None)   # NDArray[float64] of length omega + 1, expected deaths each year
```

## Sub-annual and fractional-age probabilities

When either the starting age is fractional or a payment frequency $m > 1$ is requested,
the single-period probability is derived from the ratio of adjacent $l_x$ values.

For an integer starting age $x$ and period $1/m$:

$$
{}_{1/m}q_x = 1 - \frac{l_{x + 1/m}}{l_x}
$$

For a fractional starting age $x + s$ ($0 \le s < 1$) and period $1/m$:

$$
{}_{1/m}q_{x+s} = 1 - \frac{l_{x + s + 1/m}}{l_{x + s}}
$$

Both $l_x$ values required by the ratio are drawn from the survival function — applying
the [interpolation method](#interpolating-lx-at-fractional-ages) described below when the
age is fractional, or a direct table lookup when the age is an integer — and then
**rounded to `config.decimals.lx` before the ratio is formed**.
This ensures that the ratio is computed from values that match the precision of the
public `lx()` output, keeping results self-consistent.  The final result is rounded
to `config.decimals.qx`.

```python
from lactuca import LifeTable

lt = LifeTable("PASEM2020_Rel_1o", "m")
print(lt.qx(65, m=12))    # monthly mortality probability at integer age 65
print(lt.qx(65.5))        # annual mortality probability at fractional age 65.5
print(lt.qx(65.5, m=12))  # monthly mortality probability at fractional age 65.5
```

For an **integer** starting age, the $l_x$ ratio further simplifies depending on the
interpolation assumption.  Under **UDD** (`"linear"`), linear interpolation gives:

$$
l_{x+1/m} = l_x + \frac{l_{x+1} - l_x}{m} = l_x - \frac{d_x}{m} = l_x\left(1 - \frac{q_x}{m}\right)
$$

Substituting into the ratio:

$$
{}_{1/m}q_x = \frac{q_x}{m}, \qquad {}_{1/m}p_x = 1 - \frac{q_x}{m}
$$

Under **CFM** (`"exponential"`), geometric interpolation gives $l_{x+1/m} = l_x \, p_x^{1/m}$, so:

$$
{}_{1/m}q_x = 1 - p_x^{\,1/m}, \qquad {}_{1/m}p_x = p_x^{\,1/m}
$$

See {doc}`lx_interpolation` for the derivation of both results.

:::{note}
For integer $x$ with $m = 1$, Lactuca uses the direct-lookup path described above
because it is faster and requires no $l_x$ interpolation.  Numerically the two paths
agree to within a rounding epsilon for normal mortality rates.
:::

(interpolating-lx-at-fractional-ages)=
## Interpolating lx at fractional ages

To evaluate $l_{x+s}$ at a fractional age ($0 < s < 1$), the library interpolates
between the two surrounding integer-age tabular values $l_x$ and $l_{x+1}$ using the
assumption selected by `config.lx_interpolation`.

### UDD — Uniform Distribution of Deaths (`"linear"`)

The default assumption.  Deaths within the year $[x, x+1)$ are distributed uniformly,
so $l_{x+s}$ decreases **linearly** between $l_x$ and $l_{x+1}$:

$$
l_{x+s} = l_x + (l_{x+1} - l_x) \cdot s = l_x - s \cdot d_x, \qquad 0 \le s < 1
$$

Under UDD, the fractional survival and mortality probabilities simplify to:

$$
{}_s p_x = 1 - s \cdot q_x, \qquad {}_s q_x = s \cdot q_x
$$

and the force of mortality at fractional age $x + s$ is:

$$
\mu_{x+s} = \frac{q_x}{1 - s \cdot q_x}
$$

### Constant Force of Mortality (`"exponential"`)

The force of mortality $\mu_x$ is assumed constant throughout $[x, x+1)$, giving
**geometric (log-linear)** interpolation of $l_x$:

$$
l_{x+s} = l_x \cdot \left(\frac{l_{x+1}}{l_x}\right)^s = l_x \cdot p_x^{\,s},
\qquad 0 \le s < 1
$$

Under CFM:

$$
{}_s p_x = p_x^{\,s} = e^{-\mu_x \cdot s}, \qquad \mu_x = -\ln p_x = \text{const on } [x, x+1)
$$

### Selecting the assumption

```python
from lactuca import config

config.lx_interpolation = "linear"       # UDD — linear interpolation (default)
config.lx_interpolation = "exponential"  # constant force of mortality
config.reset()                           # restore default
```

See {doc}`lx_interpolation` for a detailed comparison including derived survival formulas
and force-of-mortality behaviour.

(multi-year-survival-and-mortality)=
## Multi-year survival and mortality

For a term of $t$ years (integer or fractional), the $t$-year survival probability is
the ratio of survivors at the two boundary ages:

$$
{}_t p_x = \frac{l_{x+t}}{l_x}
$$

Both $l_{x+t}$ and $l_x$ are read from the survival function — interpolated when $x$ or
$x + t$ is fractional, looked up directly otherwise.  Each is rounded to
`config.decimals.lx` before the ratio is formed.  The result is rounded to
`config.decimals.tpx`.

The $t$-year mortality probability follows directly:

$$
{}_t q_x = 1 - {}_t p_x
$$

rounded to `config.decimals.tqx`.

```python
from lactuca import LifeTable

lt = LifeTable("PASEM2020_Rel_1o", "m")
print(lt.tpx(65, t=10))     # 10-year survival probability from integer age 65
print(lt.tqx(65, t=10))     # 10-year mortality probability from integer age 65
print(lt.tpx(65.5, t=0.5))  # 6-month survival probability from fractional age 65.5
```

:::{note}
When `t=1` and all starting ages are integers, `tpx(x, t=1)` and `px(x)` return
**identical values**: both retrieve the survival probability directly from the underlying
decrement array, rounded to `config.decimals.px`.  This guarantees exact numerical
consistency between the two methods for the most common case.  For any other value
of `t`, or for fractional starting ages, the general $l_x$-ratio path is used and
results are rounded to `config.decimals.tpx`.

`tqx` does **not** have this fast path: `tqx(x, t=1)` always uses the $l_x$-ratio and
rounds to `config.decimals.tqx`.  If `decimals.px` and `decimals.tqx` are configured
to the same value (the default), `tpx(x, t=1)` and `1 - tqx(x, t=1)` will agree to
all displayed decimal places.  If they differ, results may disagree in the last digit.
:::

## Deaths column

The expected number of deaths between consecutive integer ages is:

$$
d_x = l_x - l_{x+1}
$$

Both tabular $l_x$ values come from the rounded survival function, and the result is
rounded to `config.decimals.dx`.

```python
from lactuca import LifeTable

lt = LifeTable("PASEM2020_Rel_1o", "m")
print(lt.dx(65))   # expected deaths between exact ages 65 and 66
```

## Boundary at the terminal age

Every table defines a terminal age $\omega$: the last age index stored in the
file.  For life tables, $q_\omega = 1.0$ is required at build and load time
(see {doc}`building_tables`).  `LifeTable.qx(omega)` therefore returns $1.0$ for
integer annual frequency ($m = 1$).

Beyond the table limit the query API applies:

- $l_x = 0$ for any $x > \omega$ — ages beyond the table range are out of scope.
- ${}_{t}p_x = 0$ and ${}_{t}q_x = 1$ whenever $x + t \ge \omega$ — any interval
  that extends to or past $\omega$ has zero survival probability and certain death.

These extrapolation rules arise from the $l_x$-ratio formula and masked division when
$l_{x+t} = 0$.

```python
from lactuca import LifeTable

lt = LifeTable("PASEM2020_Rel_1o", "m")
print(lt.omega)                     # terminal age omega
print(lt.lx(lt.omega))             # 0.0: no survivors at omega
print(lt.tpx(lt.omega - 5, t=10))  # 0.0: interval extends past omega
print(lt.tqx(lt.omega - 5, t=10))  # 1.0: certain death within the interval
```

## Rounding boundaries

Rounding is applied only at public API output points.  Intermediate calculations use
full float64 precision (continuous modes) or $l_x$ values rounded to `decimals.lx`
(discrete modes).

| Method | Rounding applied |
|--------|------------------|
| `lx(x)` | `config.decimals.lx` |
| `dx(x)` | `config.decimals.dx` |
| `qx(x)` | `config.decimals.qx` |
| `px(x)` | `config.decimals.px` |
| `tpx(x, t)` | `config.decimals.tpx` (\*) |
| `tqx(x, t)` | `config.decimals.tqx` |

(\*) Exception: `tpx(x, t=1)` with all-integer ages uses `config.decimals.px` (delegates to `px()`).
See the note in the [Multi-year survival](#multi-year-survival-and-mortality) section above.

:::{note}
For sub-annual or fractional-age queries, both $l_x$ operands are rounded to
`config.decimals.lx` before the ratio is formed.  In continuous calculation modes
(`"continuous_precision"` and `"continuous_simplified"`), the high-precision $l_x$
values are used throughout and no intermediate rounding is applied to them.
:::

See {doc}`decimals_rounding` for how to configure the precision of each output type.

## See also

- {doc}`lx_interpolation` — UDD and CFM formulas in detail, including force of mortality
- {doc}`decimals_rounding` — configuring rounding precision for each output type
- {doc}`numerical_precision` — how continuous modes use unrounded lx values
- {doc}`calculation_modes` — choosing between discrete and continuous computation
- {doc}`modifying_decrements` — adjusting mortality rates before derivation (`table_combination`, `combination_mode`)

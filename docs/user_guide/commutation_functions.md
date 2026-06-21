# Commutation Functions

Commutation functions are pre-computed, age-indexed quantities that simplify actuarial present-value
calculations by absorbing discount factors and survival probabilities into reusable building blocks.
They are available both as **methods on `LifeTable`** (`lt.Nx(65)`) and as **standalone functions**
(`Nx(lt, 65)`) importable directly from `lactuca`, making them easy to use in pipelines and custom workflows.
They are independent of the calculation engine: the engine methods (`äx`, `ax`, `Ax`, etc.) do not
use commutation functions internally.

## Integer-age requirement

**All classical commutation functions — `Dx`, `Nx`, `Sx`, `Cx`, `Mx`, `Rx`, `Lx`, `Tx`,
and `ex` — accept only integer ages.**  Passing a non-integer age raises `ValueError`
immediately; no silent rounding is performed.

This is a structural consequence of their grid-based definition: suffix sums (`Nx`, `Mx`,
`Sx`, `Rx`, `Tx`) are pre-computed over the integer-age nodes of the life table.

For fractional starting ages, three continuous variants are available —
`Lx_continuous`, `Tx_continuous`, and `ex_continuous` — described in
{ref}`ex-fractional` below.

## Independence from calculation settings

Commutation functions are **not affected by `config.calculation_mode`** or (for the
integer-age methods) by **`config.lx_interpolation`**.

They are pre-computed directly from the integer-age `lx` array of the life table when
first requested, without going through the calculation engine.  `config.calculation_mode`
governs only the engine methods (`äx`, `ax`, `Ax`, `nEx`, etc.), not the commutation
cache.  Similarly, `config.lx_interpolation` only affects evaluations at fractional ages
(i.e., the `_continuous` variants and engine methods that evaluate $l_x$ at fractional
ages); it has no influence on the pre-built integer-age arrays used by the classical
commutation functions.

## Dependency chain

Each commutation symbol is an array indexed by integer age, derived in a fixed order
from the life table.  The diagram below shows the derivation chain and the standard actuarial
quantities each pair produces:

<div style="font-family: monospace; line-height: 1.8; padding: 0.5em 0;">
<table style="border-collapse: collapse; width: 100%;">
<tr>
  <td style="padding: 4px 8px; border: 1px solid #ccc; background: #f0f4ff; text-align: center;"><strong>l<sub>x</sub></strong><br/><small>life table</small></td>
  <td style="padding: 4px 8px; text-align: center;">→</td>
  <td style="padding: 4px 8px; border: 1px solid #ccc; background: #e8f5e9; text-align: center;"><strong>D<sub>x</sub></strong> = v<sup>x</sup> · l<sub>x</sub></td>
  <td style="padding: 4px 8px; text-align: center;">→</td>
  <td style="padding: 4px 8px; border: 1px solid #ccc; background: #e8f5e9; text-align: center;"><strong>N<sub>x</sub></strong> = ΣD<sub>k</sub></td>
  <td style="padding: 4px 8px; text-align: center;">→</td>
  <td style="padding: 4px 8px; border: 1px solid #ccc; background: #e8f5e9; text-align: center;"><strong>S<sub>x</sub></strong> = ΣN<sub>k</sub></td>
  <td style="padding: 4px 16px; text-align: left;"><strong>→ ä<sub>x</sub></strong> = N<sub>x</sub>/D<sub>x</sub></td>
</tr>
<tr>
  <td style="padding: 4px 8px; border: 1px solid #ccc; background: #f0f4ff; text-align: center;"><strong>d<sub>x</sub></strong><br/><small>= l<sub>x</sub> − l<sub>x+1</sub></small></td>
  <td style="padding: 4px 8px; text-align: center;">→</td>
  <td style="padding: 4px 8px; border: 1px solid #ccc; background: #fff3e0; text-align: center;"><strong>C<sub>x</sub></strong> = v<sup>x+α</sup> · d<sub>x</sub><br/><small>α from <code>mortality_placement</code></small></td>
  <td style="padding: 4px 8px; text-align: center;">→</td>
  <td style="padding: 4px 8px; border: 1px solid #ccc; background: #fff3e0; text-align: center;"><strong>M<sub>x</sub></strong> = ΣC<sub>k</sub></td>
  <td style="padding: 4px 8px; text-align: center;">→</td>
  <td style="padding: 4px 8px; border: 1px solid #ccc; background: #fff3e0; text-align: center;"><strong>R<sub>x</sub></strong> = ΣM<sub>k</sub></td>
  <td style="padding: 4px 16px; text-align: left;"><strong>→ A<sub>x</sub></strong> = M<sub>x</sub>/D<sub>x</sub></td>
</tr>
</table>
</div>


(ex-complete-vs-curtate)=
## `ex` — complete expectation (not curtate)

`LifeTable.ex(x)` returns the **complete** expectation of future lifetime
$\mathring{e}_x = T_x / l_x$ under the configured `lx_interpolation` (UDD gives the
trapezoidal person-years formula; CFM uses integration over exponential within-year
survival). Classical **curtate** $e_x = \sum_{k=1}^{\omega-x} {}_kp_x$ is not exposed
as a separate public method — use commutation ratios or build the sum explicitly if
curtate notation is required.

## Why $\ddot{a}_x = N_x/D_x$

The whole-life annuity-due pays 1 at the start of each policy year while $(x)$ is alive.
Its actuarial present value is:

$$
\ddot{a}_x = \sum_{k=0}^{\omega-x} v^k \cdot {}_kp_x
= \sum_{k=0}^{\omega-x} v^k \cdot \frac{l_{x+k}}{l_x}
$$

Multiplying and dividing each term by $v^x$ and recognising $D_{x+k} = v^{x+k}\,l_{x+k}$:

$$
\ddot{a}_x = \frac{1}{v^x l_x}\sum_{k=0}^{\omega-x} D_{x+k} = \frac{N_x}{D_x}
$$

An analogous argument gives $A_x = M_x/D_x$: replace $v^k\,{}_kp_x$ with
$v^{k+1}\,{}_kp_x\,q_{x+k} = C_{x+k}/l_x$ in each summand.

## Definitions

### Dx — Discounted survivors

$$D_x = v^x \cdot l_x$$

The present value of $l_x$ units payable at age $x$ from the life table origin.

### Nx — Sum of Dx

$$N_x = \sum_{k=x}^{\omega} D_k = D_x + D_{x+1} + \cdots + D_{\omega}$$

Used in whole-life annuity-due: $\ddot{a}_x = N_x / D_x$.

### Sx — Sum of Nx

$$S_x = \sum_{k=x}^{\omega} N_k$$

Used for increasing annuities.

### Cx — Discounted deaths

$$C_x = v^{x+1} \cdot d_x = v^{x+1} \cdot (l_x - l_{x+1})$$

The present value of expected deaths in $[x, x+1)$ discounted to the origin.
The factor $v^{x+1}$ reflects the convention that deaths occur at the
**end** of the year of death (annual discrete insurance).

### Mx — Sum of Cx

$$M_x = \sum_{k=x}^{\omega} C_k$$

Used in whole-life insurance: $A_x = M_x / D_x$.

### Rx — Sum of Mx

$$R_x = \sum_{k=x}^{\omega} M_k$$

Used for increasing insurance.

### Lx — Person-years lived

`Lx(x)` takes no `ir` parameter (no discounting involved).

Under the Uniform Distribution of Deaths (UDD) assumption, the person-years lived
in $[x, x+1)$ are:

$$L_x = l_x \left(1 - \frac{q_x}{2}\right) = \frac{l_x + l_{x+1}}{2}$$

By convention, $L_\omega = 0$.

### Tx — Total future lifetime

`Tx(x)` takes no `ir` parameter.

$$T_x = \sum_{k=x}^{\omega-1} L_k$$

The total number of person-years lived by the cohort from age $x$ to $\omega$.

### ex — Complete life expectancy

`ex(x)` takes no `ir` parameter.

$$\mathring{e}_x = \frac{T_x}{l_x}$$

Returns the **complete** life expectancy under the UDD approximation, not the
curtate expectation $e_x = \sum_{k=1}^{\infty} {}_kp_x$ based on annual survival steps.

(ex-fractional)=
## Continuous variants for fractional ages

The three demographic functions (`Lx`, `Tx`, `ex`) each have a `_continuous` counterpart
for fractional ages.  These use the trapezoidal rule with `m` sub-intervals per year
and evaluate $l_{x+t}$ at fractional ages using the configured interpolation
assumption, so they **do** respect `config.lx_interpolation`
(unlike the integer-age methods, which read the pre-built integer-age `lx` array directly).

**Passing an integer age to a `_continuous` method raises `ValueError`** — use the
standard method for integer ages.

| Method | Computes | Raises `ValueError` if… |
|---|---|---|
| `Lx_continuous(x, m=12)` | $\int_x^{x+1} l_t\,dt$ via trapezoidal rule | $x$ is integer |
| `Tx_continuous(x, m=12)` | $T_x$ at fractional age | $x$ is integer |
| `ex_continuous(x, m=12)` | $\mathring{e}_x = T_x / l_x$ at fractional age | $x$ is integer |

```python
from lactuca import LifeTable

lt = LifeTable("PASEM2020_Rel_1o", "m")

lt.ex(65)              # ✓ integer age — UDD closed form
lt.ex_continuous(65)   # ✗ raises ValueError: use ex() for integer ages
lt.ex_continuous(65.5) # ✓ fractional age — trapezoidal rule

# Lx_continuous and Tx_continuous follow the same pattern:
lt.Lx_continuous(65.5)        # person-years in [65.5, 66.5), default m=12
lt.Lx_continuous(65.5, m=52)  # higher precision: 52 sub-intervals
lt.Tx_continuous(65.5)        # total future person-years from age 65.5
```

## Using commutation functions in Lactuca

Discounted commutation functions (`Dx`, `Nx`, `Sx`, `Cx`, `Mx`, `Rx`) use the table's
`interest_rate` by default; an explicit `ir=` argument overrides it for that call only.
If neither is set, a `ValueError` is raised.
The demographic functions (`Lx`, `Tx`, `ex`) take no interest rate:

```python
from lactuca import LifeTable

lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)

Dx = lt.Dx(65)   # D_65
Nx = lt.Nx(65)   # N_65
Cx = lt.Cx(65)   # C_65
Mx = lt.Mx(65)   # M_65
Sx = lt.Sx(65)   # S_65
Rx = lt.Rx(65)   # R_65
Lx = lt.Lx(65)   # L_65  (no discounting)
Tx = lt.Tx(65)   # T_65  (no discounting)
ex = lt.ex(65)   # e_65  (no discounting)
```

### `x0` parameter for term-structured interest rates

When using a piecewise `InterestRate` (term structure), the discount factor $v^x$ in
$D_x = v^x \cdot l_x$ is computed relative to a reference origin age.  Set `x0` to
the age of the origin (e.g., the entry age) so that the discount aligns correctly with
the term structure:

```python
from lactuca import LifeTable, InterestRate

ir_term = InterestRate(terms=[10], rates=[0.02, 0.04])
lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=ir_term)

# Discounting from age x0=25 — D_65 reflects 40 years of discounting
Dx_65 = lt.Dx(65, x0=25)
Nx_65 = lt.Nx(65, x0=25)
ax_65 = Nx_65 / Dx_65
```

The default `x0=0` is correct for constant interest rates and for term structures
measured from the cohort origin.

### Functional-style calls

All commutation functions are importable directly from `lactuca` as standalone
functions (they live in `lactuca.functional` internally).  The functional form takes
the table instance as the first argument; all remaining arguments are identical to
the method call:

```python
from lactuca import LifeTable, Dx, Nx, Cx, Mx, Sx, Rx, Lx, Tx, ex

lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)

Nx(lt, 65)          # equivalent to lt.Nx(65)
Nx(lt, 65) / Dx(lt, 65)  # whole-life annuity-due via commutation
ex(lt, 65)          # complete life expectancy
```

This form is particularly useful when composing commutation functions in pipelines
or passing them as callbacks.  See {doc}`functional_api` for the full reference.

## Commutation formulas vs. direct engine methods

An important distinction: **Lactuca's annuity and insurance methods (`äx`, `ax`, `Ax`,
`nEx`, etc.) do not use commutation functions internally.**  They use a direct
payment-grid summation dispatched through one of the four calculation modes
(`discrete_precision`, `discrete_simplified`, `continuous_precision`,
`continuous_simplified`).  Commutation functions are a separate, standalone tool for
classical closed-form work.

### What the direct engine does (m=1, integer age)

For `äx(65, ir=0.03)` under `discrete_precision` with `m=1`, the engine builds the
payment grid $t = 0, 1, 2, \ldots, \omega - 65$ and computes:

$$\ddot{a}_{65} = \sum_{t=0}^{\omega-65} v^t \cdot {}_tp_{65}
= \frac{1}{l_{65}} \sum_{t=0}^{\omega-65} v^t \cdot l_{65+t}$$

No commutation cache is accessed.  Every term is kept at full float64 precision;
**only the final sum is rounded**.

### Numerical equivalence for m=1, integer ages

For integer $x$ and $m=1$, the formula above is mathematically identical to
$N_x / D_x$ — both equal $\frac{\sum_t v^t l_{x+t}}{l_x}$.  A small numerical
difference (typically of order $10^{-6}$ to $10^{-8}$ at the default 10-decimal
commutation precision) arises because each $D_k = v^k \cdot l_k$ is individually
rounded before being accumulated into $N_x$, and $N_x$ itself is stored rounded.
The engine accumulates unrounded float64 terms and rounds only the final result.

This cross-check should pass with a loose tolerance:

```python
from lactuca import LifeTable

lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)

a_comm   = lt.Nx(65) / lt.Dx(65)   # commutation path
a_direct = lt.äx(65)               # direct engine (discrete_precision)
print(abs(a_comm - a_direct) < 1e-6)  # True
```

### Divergence for m > 1

For $m > 1$, the two approaches diverge conceptually, not just numerically.
Lactuca's engine evaluates exact sub-annual survival directly; classical commutation
functions do not have a native $m > 1$ form without separate $m$-thly tables (which
are not precomputed in Lactuca):

| Approach | How $\ddot{a}^{(12)}_{65}$ is computed |
|---|---|
| `discrete_precision`, `m=12` | Exact summation at $t = 0, \tfrac{1}{12}, \tfrac{2}{12}, \ldots\,$; evaluates $l_{65+t}$ at each sub-annual point using the configured interpolation (respects `config.lx_interpolation`) |
| `discrete_simplified`, `m=12` | Woolhouse 2-term on integer years: $\ddot{a}^{(12)}_{x:\overline{n}|} \approx \ddot{a}_{x:\overline{k}|} - \tfrac{m-1}{2m}$ with $k=\lfloor n\rfloor$; when $n$ is fractional and $m>1$, an exact $m$-thly tail on $(k,n]$ is added (hybrid $k+s$ — see {doc}`calculation_modes`) |
| Classical commutation, $m>1$ | Requires separate $m$-thly commutation tables — not provided in Lactuca; approximated by the Woolhouse formula |

For $m > 1$, use the engine methods (`äx`, `ax`, etc.) with the appropriate `m` and
`calculation_mode`.

### Continuous calculation modes

`continuous_precision` and `continuous_simplified` use integral-based formulas over the
entire future lifetime and are entirely outside the scope of classical commutation
functions, which are inherently annual and discrete.

### When to use each approach

| Scenario | Recommended tool |
|---|---|
| Classical closed-form work, `m=1`, integer ages | Commutation functions |
| Cross-checks and custom actuarial formulas | Commutation functions (verify against engine) |
| Sub-annual payments, `m > 1` | `äx()`, `ax()` with `discrete_precision` |
| Continuous annuities / insurances | `äx()`, `ax()` with `continuous_precision` |
| Fractional starting age (any `m`) | Engine methods — commutation functions do not apply |
| Production valuations (all cases) | Engine methods (`äx`, `ax`, `Ax`, `nEx`, …) |

## Key actuarial formulas using commutation functions

### Whole-life annuity-due

$$\ddot{a}_x = \frac{N_x}{D_x}$$

### Temporary annuity-due

$$\ddot{a}_{x:\overline{n}|} = \frac{N_x - N_{x+n}}{D_x}$$

### Whole-life insurance

$$A_x = \frac{M_x}{D_x}$$

### Term insurance

$$A^1_{x:\overline{n}|} = \frac{M_x - M_{x+n}}{D_x}$$

### Pure endowment

$${}_{n}E_x = \frac{D_{x+n}}{D_x}$$

### Increasing insurance

$$IA_x = \frac{R_x}{D_x}$$

## Notes

- All results are rounded to the corresponding precision setting — `Config.decimals.Dx`,
  `.Nx`, `.Cx`, `.Mx`, etc. (default: 10 decimal places).
- When a method is called without an age argument (e.g. `lt.Nx()`), the full array
  from age 0 to $\omega$ is returned.
- Commutation functions are **not affected by `config.calculation_mode`** or
  `config.lx_interpolation`.  The `_continuous` variants (`Lx_continuous`, `Tx_continuous`,
`ex_continuous`) are the sole exception: they evaluate `lx` at fractional ages at each
integration point and therefore do respect `config.lx_interpolation`.

## See also

- {doc}`calculation_modes` — the four modes and how each evaluates annuities and insurances
- {doc}`lx_interpolation` — UDD vs. CFM, and when `config.lx_interpolation` applies
- {doc}`interest_rates_guide` — setting the interest rate
- {doc}`tables_taxonomy` — table types, column structure and decrement classes

## `Lx`, `Tx`, and `lx_interpolation`

Integer-age `Lx` and `Tx` use the UDD fast path ($L_x = l_x - d_x/2$) only when
`config.lx_interpolation = "linear"`. With `"exponential"` (constant force of mortality
within each year), the same methods integrate over within-year survival and may differ
from the UDD shortcut. Engine methods honour `lx_interpolation` at fractional ages in
all calculation modes.


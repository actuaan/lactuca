# lx Interpolation

Life table calculations often require the number of survivors $l_{x+s}$ at a **fractional age**
$x + s$ where $0 < s < 1$.  Two assumptions bridging integer-age $l_x$ values are
supported in Lactuca.

## Interpolation assumptions

### 1. UDD — Uniform Distribution of Deaths

The most commonly used assumption in discrete actuarial models.

$$l_{x+s} = (1-s)\,l_x + s\,l_{x+1} = l_x - s\,d_x$$

This is equivalent to assuming that deaths within $[x, x+1)$ are uniformly spread over the year.

**Survival probability** under UDD:

$$
{}_s p_x = \frac{l_{x+s}}{l_x} = \frac{l_x - s\,d_x}{l_x} = 1 - s\,\frac{d_x}{l_x} = 1 - s\,q_x
$$

**Force of mortality** under UDD:

$$
\mu_{x+s} = \frac{q_x}{1 - s\,q_x}
$$

### 2. Constant force of mortality

$$
l_{x+s} = l_x \cdot \left(\frac{l_{x+1}}{l_x}\right)^s = l_x \cdot p_x^s
$$

Equivalent to assuming $\mu_{x+s} = -\ln p_x$ constant throughout $[x, x+1)$.

## Configuration in Lactuca

The interpolation method is selected via `config.lx_interpolation`. The default is `"linear"` (UDD).
See {doc}`configuration` for the full settings reference.

```python
from lactuca import config

config.lx_interpolation = "linear"       # UDD — linear interpolation (default)
config.lx_interpolation = "exponential"  # constant force of mortality
```

## Fractional survival probabilities

For payment frequencies $m > 1$, Lactuca needs ${}_{{1/m}}p_x = p_x(m)$.  With UDD:

$$
{}_{1/m} p_x = 1 - \frac{q_x}{m}
$$

With constant force:

$$
{}_{1/m} p_x = e^{-\mu_x / m} = p_x^{1/m}
$$

Call `lt.px(x, m=m)` or `lt.qx(x, m=m)` to obtain these values; Lactuca applies the
configured interpolation assumption automatically.  These methods are also importable
directly from `lactuca` as standalone functions (`px(lt, x, m=m)`, `qx(lt, x, m=m)`).

## Example: monthly survival

```python
from lactuca import LifeTable, px, qx

lt = LifeTable("PASEM2020_Rel_1o", "m")

# Annual survival
lt.px(65)         # p_65 = 1 - q_65

# Monthly survival under UDD
lt.px(65, m=12)   # 1 - q_65/12  (exact under UDD)

# Functional-style equivalents (importable directly from lactuca)
px(lt, 65)        # same as lt.px(65)
px(lt, 65, m=12)  # same as lt.px(65, m=12)
qx(lt, 65, m=12)  # same as lt.qx(65, m=12)
```

## Example: fractional starting age

With a fractional starting age, UDD and CFM produce different $l_{x+s}$ values.
The difference is small for low $q_x$ but grows at older ages where mortality is higher.

```python
from lactuca import LifeTable, config

# discrete_precision: payment grid at 66.5, 67.5, … → lx_interpolation applies at every step
config.calculation_mode = "discrete_precision"
lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)

# --- UDD (linear interpolation) ---
config.lx_interpolation = "linear"

lx_udd  = lt.lx(65.5)   # l_65 - 0.5 * d_65  (linear interpolation)
px_udd  = lt.px(65.5)    # survival from age 65.5 to 66.5  under UDD
ax_udd  = lt.ax(65.5)    # annuity-immediate for age 65.5 under UDD

# --- Constant force of mortality (exponential interpolation) ---
config.lx_interpolation = "exponential"

lx_cfm  = lt.lx(65.5)   # l_65 * (l_66 / l_65)^0.5  (geometric interpolation)
px_cfm  = lt.px(65.5)    # survival from age 65.5 to 66.5 under CFM
ax_cfm  = lt.ax(65.5)    # annuity-immediate for age 65.5 under CFM

# CFM gives slightly lower lx (geometric mean < arithmetic mean when l is decreasing),
# and therefore slightly lower survival and annuity values than UDD.
print(lx_udd > lx_cfm)   # True
print(ax_udd > ax_cfm)   # True (typically for adult ages)
```

## Continuous calculations

`config.lx_interpolation` affects the two continuous calculation modes differently:

- **`continuous_precision`** — evaluates annuity and insurance values by numerical integration
  over a fine time grid.  At each grid point $t$, the fractional survival $l_{x+t}$ is computed
  using the configured interpolation assumption at full float64 precision, without intermediate
  rounding.  This is where the difference between UDD and constant force has the most impact,
  since many fractional-age evaluations are performed per calculation.

- **`continuous_simplified`** — approximates continuous annuities by averaging an annual annuity-due
  and an annuity-immediate, using only integer payment steps.  When the starting age `x` is an
  integer and all other parameters are integers, only integer-age $l_x$ values are evaluated and
  `config.lx_interpolation` has no effect.  However, as with any other mode, a fractional
  starting age causes lx to be evaluated at fractional ages at every step, so
  `config.lx_interpolation` applies in that case.

See {doc}`calculation_modes` for details on when each continuous mode is used.

## Scope of effect

The rule is simple: `config.lx_interpolation` applies whenever $l_{x+t}$ must be evaluated at
a **fractional age** — that is, whenever $x + t$ is non-integer.  This occurs in three
situations:

1. **Fractional starting age** (e.g. `lt.ax(35.123, ...)`) — every calculation mode must
   evaluate $l_{35.123}$, $l_{36.123}$, $l_{37.123}$, …, so `lx_interpolation` always takes
   effect regardless of `m` or `calculation_mode`.

2. **Payment frequency `m > 1` with `discrete_precision`** — the exact payment grid contains
   points at $x + k/m$ for $k = 1, 2, \ldots$, which are fractional for integer $x$ and $m > 1$.

3. **`continuous_precision` mode** — the integration grid is always dense with fractional
   time points, so fractional-age lx evaluation is structurally unavoidable.

Two modes structurally avoid fractional lx evaluation when the starting age is integer,
regardless of `m`:

- **`discrete_simplified`** — the Woolhouse approximation resolves sub-annual payments
  algebraically from two annual-step annuities, without ever requesting $l_x$ at a
  sub-annual grid point.
- **`continuous_simplified`** — ignores `m` internally and operates on annual-step
  annuities (due and immediate), so with integer starting age only integer-age $l_x$
  values are ever evaluated.

In both cases, `config.lx_interpolation` has no effect when the starting age is integer.

## Numerical comparison: UDD vs CFM

The two assumptions produce indistinguishable results for most standard valuations, but
their difference becomes visible when payment frequency is high or when annuity
calculations involve a fractional starting age.  The differences arise because UDD
places slightly more weight on deaths near the middle of the year, while CFM distributes
them geometrically.

The code below compares whole-life annuity values $\ddot{a}_x^{(m)}$ for a male life
using `PASEM2020_Rel_1o` at $i = 3\%$, at three representative ages and three payment
frequencies.  Since `config.lx_interpolation` is evaluated at call-time, both valuations
use the same table instance with the setting switched between calls:

```python
from lactuca import LifeTable, config

config.decimals.annuities = 6

lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)
# config.lx_interpolation is read at call-time, not at table construction

ages = [45, 65, 80]
freqs = [1, 12, 52]

print(f"{'Age':>4}  {'m':>4}  {'UDD (linear)':>14}  {'CFM (exp)':>14}  {'diff':>10}")
for x in ages:
    for m in freqs:
        config.lx_interpolation = "linear"
        a_udd = lt.äx(x, m=m)
        config.lx_interpolation = "exponential"
        a_cfm = lt.äx(x, m=m)
        diff  = a_cfm - a_udd
        print(f"{x:>4}  {m:>4}  {a_udd:>14.6f}  {a_cfm:>14.6f}  {diff:>+10.6f}")

config.reset()
```

**Key observations:**

- At $m = 1$ (annual) the two assumptions give identical results for integer starting
  ages — no fractional-age evaluation is ever performed.
- For $m = 12$ (monthly) and above, the difference is non-zero but small (typically
  well below one per-mille per unit benefit at standard ages and interest rates).
- At higher ages the difference is slightly larger because the mortality gradient within
  each year is steeper — UDD and CFM diverge more when deaths are far from uniformly
  distributed within the year.
- CFM consistently yields **slightly lower** annuity values than UDD for monthly and
  higher frequencies: geometric interpolation is concave, placing fewer expected
  survivors at sub-annual checkpoints than the uniform distribution assumed by UDD.

For **regulatory or pricing use**, the difference is negligible for $m \le 12$.
For **continuous modes** (`continuous_precision`) the distinction matters more; see
{doc}`calculation_modes`.

## See also

- {doc}`configuration` — `config.lx_interpolation` setting reference
- {doc}`calculation_modes` — discrete vs. continuous mode
- {doc}`qx_derivation_flow` — how the life table's lx array is built from raw qx values
- {doc}`numerical_precision` — rounding policy and numerical precision in life table calculations

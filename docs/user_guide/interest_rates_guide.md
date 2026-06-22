# Interest Rates

This guide covers the `InterestRate` class — how to create, configure, and use interest rate
objects in all actuarial calculations.

## Creating an InterestRate

### Constant rate

A constant `InterestRate` represents a flat term structure — the same effective annual rate
applies to all durations.

```python
from lactuca import InterestRate

ir = InterestRate(0.03)   # 3% per annum, effective annual
print(ir.summary())       # InterestRate: constant rate = 0.030000
```

### Piecewise (term structure)

Pass a list of `terms` (in years) and `rates`.  The `rates` list must have one more element
than `terms` (the final rate applies indefinitely):

```python
from lactuca import InterestRate

# 3% for the first 5 years, 3.5% for the next 5, 4% thereafter
ir = InterestRate(terms=[5, 5], rates=[0.03, 0.035, 0.04])
print(ir.summary())   # InterestRate: piecewise terms=[5. 5.], rates=[0.03  0.035 0.04 ]
```

Segment durations (`terms`) can be any positive real number — fractional years are fully supported:

```python
from lactuca import InterestRate

# 2% for 5.25 yr, 3% for 2.5 yr, 3.5% for 3 yr, 4% thereafter
ir = InterestRate(terms=[5.25, 2.5, 3], rates=[0.02, 0.03, 0.035, 0.04])
print(ir.summary())   # InterestRate: piecewise terms=[5.25 2.5  3.  ], rates=[0.02  0.03  0.035 0.04 ]
```

### Term units

By default, `terms` are interpreted in years.  Pass `term_unit` to specify another unit —
`"months"`, `"weeks"`, or `"days"`.  Lactuca converts to years internally using the
`days_per_year` (or `weeks_per_year`) setting from `Config`:

```python
from lactuca import InterestRate

# 6 months at 1%, 12 months at 1.2%, 1.3% thereafter
ir = InterestRate(terms=[6, 12], rates=[0.01, 0.012, 0.013], term_unit="months")
print(round(ir.vn(1.5), 4))   # discount factor for 1.5 years

# 90 days at 2%, 2.5% thereafter
ir = InterestRate(terms=[90], rates=[0.02, 0.025], term_unit="days")
print(ir.summary())   # InterestRate: piecewise terms=[0.246407], rates=[0.02  0.025]  (terms shown in years)
```

### Scenarios

Store multiple interest rate scenarios under named keys:

```python
ir = InterestRate({
    "base":    0.03,
    "stress":  ([5, 5], [0.02, 0.03, 0.04]),
    "optimist": 0.04,
})
print(ir.summary())   # lists all scenarios and the active one

# Activate a scenario
ir.active_scenario = "stress"
print(round(ir.vn(10), 4))   # uses stress scenario
```

### Copying and snapshotting

Use `ir.copy()` to take an independent deep copy before switching scenarios, passing
the curve to a function that may mutate it, or building stress variants from a shared
baseline:

```python
from lactuca import InterestRate

ir_base = InterestRate({"base": 0.03, "stress": ([5, 5], [0.02, 0.03, 0.04])})

ir_a = ir_base.copy()
ir_b = ir_base.copy()

ir_a.active_scenario = "base"
ir_b.active_scenario = "stress"

print(round(ir_a.vn(10), 4))   # 0.7441  (base)
print(round(ir_b.vn(10), 4))   # 0.7014  (stress)
# ir_base is untouched
```

`ir.copy()` uses `copy.deepcopy` internally; the `Config` singleton is shared
(not duplicated), and a fresh thread lock is created for the copy.

(interest-rate-scenarios-lifetable)=
### Scenarios with LifeTable and batch methods

Multi-scenario `InterestRate` containers are passed **by reference** to `LifeTable`
and to every `ir=` argument.  Each annuity, insurance, endowment, or commutation call
reads the **currently active scenario at call time** — the scenario is **not**
snapshotted when you construct `LifeTable` or when you build a per-policy list such as
`ir=[ir, ir, ir]`.

:::{important}
**Reference semantics.**  `lt.interest_rate` is the same object you passed at
construction (or assigned later).  Changing `ir.active_scenario` after creating
`lt` changes subsequent results from `lt.ax()`, `lt.äx()`, and all other methods
that use the table default rate, unless you override with an explicit `ir=` for that
call only.
:::

```python
from lactuca import InterestRate, LifeTable

ir = InterestRate({
    "base":      0.02,
    "piecewise": ([5, 10], [0.01, 0.02, 0.025]),
})
lt = LifeTable("PASEM2020_Gen_2o", "m", interest_rate=ir)

ir.active_scenario = "base"
a_base = lt.ax(65, n=20)

ir.active_scenario = "piecewise"
a_piecewise = lt.ax(65, n=20)   # differs from a_base
```

To **freeze** the scenario active when you attach a rate to a table, pass a deep copy:

```python
lt = LifeTable("PASEM2020_Gen_2o", "m", interest_rate=ir.copy())
# lt.interest_rate is independent; parent ir.active_scenario switches do not affect lt
```

The same contract applies when you pass a multi-scenario container explicitly:

```python
ir.active_scenario = "base"
val = lt.ax(65, n=20, ir=ir)          # uses base
ir.active_scenario = "piecewise"
val2 = lt.ax(65, n=20, ir=ir)         # uses piecewise
```

#### Sensitivity patterns

| Pattern | When to use |
|---------|-------------|
| `for name, scen in ir.scenarios.items(): lt.äx(..., ir=scen)` | Compare scenarios explicitly; each `scen` is a simple curve |
| `ir.active_scenario = "stress"; lt.äx(...)` | One shared container; switch in place (see {doc}`../cookbook` recipe 5) |
| `interest_rate=ir.copy()` | Fix the active scenario at copy time on a `LifeTable` |
| `return_flows=True` then `ecf.dot(ir.vn(tg))` | Many discount scenarios on the same mortality cashflows (see {doc}`probable_flows`) |

`GrowthRate` multi-scenario containers follow the same call-time rule for `gr=`;
see {doc}`growth_rates_guide` — [Multi-scenario `gr=` with LifeTable](#growth-rate-scenarios-lifetable).
Batch broadcasting details: {doc}`batch_calculations` — [Multi-scenario `ir` and `gr`](#multi-scenario-ir-gr-batch).

## Discount factors

The discount factor $v^n = (1+i)^{-n}$ is the present value of one unit payable in $n$ years.
`vn(n)` computes this directly; `vx(x, x0)` computes the discount factor for any interval
$[x_0, x]$ — exactly $x - x_0$ years regardless of the absolute values of $x$ and $x_0$.
Both methods are fully vectorised and support constant and piecewise curves.

:::{note}
`vn(n)` raises `ValueError` if the rate satisfies $i \leq -1$, because $(1+i)^{-n}$
requires $(1+i) > 0$.  Rates in $(-1, 0)$ are valid (e.g. negative ECB deposit rates)
and do not raise.  See {ref}`rate--1`.
:::

```python
from lactuca import InterestRate

ir = InterestRate(0.03)

# v^n: discount factor for a duration of n years
print(round(ir.vn(5), 4))              # 0.8626
print(ir.vn([1, 5, 10]).round(4))      # [0.9709 0.8626 0.7441]

# vx(x, x0): discount factor from x0 to x
print(round(ir.vx(10, x0=0), 4))       # 0.7441  (10-year discount)
print(round(ir.vx(15, x0=5), 4))       # 0.7441  (same: only 10 years elapse)
```

## Annuities on `InterestRate`

`InterestRate` also exposes its own annuity methods for pure-financial (non-mortality) calculations:

```python
from lactuca import InterestRate

ir = InterestRate(0.03)

print(round(ir.a(n=10, m=4), 4))   # 8.6256   certain annuity-immediate, 10 yr, quarterly
print(round(ir.ä(n=10, m=4), 4))  # 8.6896   certain annuity-due,       10 yr, quarterly
```

These are the pure-interest building blocks used internally by `LifeTable` annuity methods.

### Batch mode

Both methods support **batch inputs**: passing any of `n`, `d`, or `ts` as a list,
tuple, or `NDArray` switches to batch mode and returns `NDArray[np.float64]`.

```python
from lactuca import InterestRate, payment_times

ir = InterestRate(0.03)

# Duration analysis — PV over a ladder of maturities
maturities = payment_times(n=30, m=1)
pv_curve   = ir.a(n=maturities)     # NDArray shape (30,)

# Per-policy deferments
pv = ir.a(n=20.0, d=[0.0, 5.0, 10.0])
print(pv)
# [14.87747486 12.8334405  11.07023851]

# Net premium (annuity-due, no mortality)
premiums = 1.0 / ir.ä(n=[10.0, 20.0, 30.0])
```

**One curve per instance:** batch `ir.a()` / `ir.ä()` always discount with the
term structure stored on that single `InterestRate` object.  You can vary `n`, `d`, `ts`,
`m`, or `gr` per policy, but not attach a different curve per row in the same call.
For mortality-inclusive BEL with heterogeneous discount curves per policy, pass a
per-policy `ir` list (or object-dtype `Series` of `InterestRate` instances) to
`LifeTable` batch methods — see {doc}`batch_calculations` (*Broadcasting rules*).

For portfolio cash flows (`return_flows=True`), benefit-weighted portfolio aggregation
(`benefits=face_values`), error handling (`on_error='nan'`), and full examples including
IFRS 17 / ALM use cases, see
{ref}`pure-financial-annuities`.

### `return_flows` on `InterestRate` vs `LifeTable`

`InterestRate.a()` and `InterestRate.ä()` support `return_flows=True` in **batch mode
for all four** calculation modes (`discrete_precision`, `discrete_simplified`,
`continuous_precision`, `continuous_simplified`).

`LifeTable` annuity, insurance, and endowment methods restrict batch
`return_flows=True` to **precision modes only** (`discrete_precision` and
`continuous_precision`); simplified modes raise `ValueError` in batch.

| API | Batch `return_flows=True` in simplified modes |
|-----|-----------------------------------------------|
| `InterestRate.a()` / `InterestRate.ä()` | Supported (all four modes) |
| `LifeTable.ax()` / `äx()` / `Ax()` / `nEx()` (and joint-life equivalents) | `ValueError` — use a precision mode |

See {doc}`batch_calculations` — *`return_flows` requires a precision calculation mode*
for the `LifeTable` contract and {ref}`pure-financial-annuities` for pure-financial
aggregate flows.

## Querying individual rates

Use `get_rate(t)` to retrieve the spot rate applying at any instant.  For piecewise curves
this returns the rate of the segment containing `t`; for constant curves it always returns
the single rate.  The call is vectorised: pass a list or array to get all rates at once.

**Segment boundaries (junction times):** piecewise segments are left-closed intervals
$[T_{k-1}, T_k]$ where $T_0 = 0$ and $T_k$ is the cumulative end of segment $k$.
At an exact junction $t = T_k$, `get_rate` returns the rate of the segment **ending**
at $T_k$ (not the next segment).  Internally this follows
`numpy.searchsorted(cum_terms, t, side='left')`, consistent with `get_segment_info`
and discount compounding in `vn()`.

```python
from lactuca import InterestRate

ir = InterestRate(terms=[5, 5], rates=[0.02, 0.03, 0.04])

print(ir.get_rate(2))                    # 0.02  (first segment)
print(ir.get_rate(3.5))                  # 0.02  (fractional t, still in first segment)
print(ir.get_rate(5))                    # 0.02  (junction: still first segment)
print(ir.get_rate(7))                    # 0.03  (second segment)
print(ir.get_rate([2, 7, 12]))           # [0.02 0.03 0.04]
print(ir.get_rate([0.5, 5.5, 10.5]))     # [0.02 0.03 0.04]  (fractional t values)
```

For the full details of the segment containing `t` — its start, end, index, and relative
position — use `get_segment_info(t)`:

```python
from lactuca import InterestRate

ir = InterestRate(terms=[5, 5], rates=[0.02, 0.03, 0.04])
info = ir.get_segment_info(7)

print(info['segment_index'])         # 1        (second segment, 0-based)
print(info['rate'])                  # 0.03
print(info['segment_start'])         # 5.0
print(info['segment_end'])           # 10.0
print(info['position_in_segment'])   # 2.0      (t=7 is 2 yr into the segment)
```

## Effective rate over an interval

For a piecewise curve, the rate in each segment is the *instantaneous* annual rate.
To obtain the single effective annual rate that is equivalent over an arbitrary
interval $[t_1, t_2]$, use `get_effective_rate`:

$$r_{\text{eff}} = \left(\frac{v(t_2)}{v(t_1)}\right)^{-1/(t_2-t_1)} - 1$$

```python
from lactuca import InterestRate

ir = InterestRate(terms=[5, 5], rates=[0.02, 0.03, 0.04])

# Effective rate from year 0 to year 10 (spans two segments)
print(round(ir.get_effective_rate(0, 10), 6))     # 0.024988  (compound, not arithmetic, average)

# Single-segment interval: equals get_rate exactly
print(round(ir.get_effective_rate(0, 5), 6))      # 0.02

# Cross-segment interval: 2 yr at 2%, 3 yr at 3%
print(round(ir.get_effective_rate(3, 8), 6))      # 0.025988

# Fractional endpoints are fully supported
print(round(ir.get_effective_rate(2.5, 7.5), 6))  # 0.024988  (2.5 yr each side of boundary)
print(round(ir.get_effective_rate(0.5, 8.5), 6))  # 0.024363  (4.5 yr at 2%, 3.5 yr at 3%)
print(round(ir.get_effective_rate(0, 6.5), 6))    # 0.022299  (5 yr at 2%, 1.5 yr at 3%)
```

This is the method to use when pricing bonds, computing forward rates, or selecting a
single representative rate for a multi-segment projection.

## Force of interest

The force of interest $\delta$ is the continuously compounded rate equivalent to the effective
annual rate $i$: $\delta = \ln(1+i)$.  For a constant-rate object, call `delta()` with no
arguments.  For piecewise curves, pass the time $t$ to select the segment; the result reflects
the instantaneous force applying at that point.

```python
from lactuca import InterestRate

ir = InterestRate(0.03)
print(round(ir.delta(), 5))   # 0.02956   δ = ln(1.03)
```

For piecewise curves, pass the time at which to evaluate the instantaneous force:

```python
from lactuca import InterestRate

ir = InterestRate(terms=[5, 5], rates=[0.02, 0.03, 0.04])

print(round(ir.delta(2), 5))            # 0.01980  (first segment:  ln 1.02)
print(round(ir.delta(7), 5))            # 0.02956  (second segment: ln 1.03)
print(ir.delta([2, 7]).round(5))        # [0.01980 0.02956]
```

When a single *average* force over $[0, n]$ is needed — for continuous annuity
approximations or IFRS 17 OCI unwinding — use `get_average_force`:

$$\bar{\delta} = -\frac{\ln v^n}{n} \qquad \text{such that } e^{-n\bar{\delta}} = v^n$$

```python
from lactuca import InterestRate

ir = InterestRate(terms=[5, 5], rates=[0.02, 0.04, 0.03])

print(round(ir.get_average_force(10), 6))   # 0.029512  (weighted across all segments)
print(round(ir.get_average_force(5), 6))    # 0.019803  (== ln(1.02), single segment)
```

For a constant rate, `get_average_force(n)` equals `delta()` for all `n`.

## Accumulation factor

The accumulation factor $s_n = (1+i)^n$ is the reciprocal of the discount factor: $s_n = 1 / v^n$.
It answers the question "how much does one unit grow to after $n$ years?"  For piecewise curves,
the factor compounds correctly across segment boundaries.

```python
from lactuca import InterestRate

ir = InterestRate(0.03)

print(round(ir.sn(5), 6))              # 1.159274  (1+i)^5, accumulation factor
print(ir.sn([1, 5, 10]).round(6))      # [1.03     1.159274 1.343916]
```

For a piecewise curve the compounding spans multiple segments automatically:

```python
from lactuca import InterestRate

ir = InterestRate(terms=[5, 5], rates=[0.02, 0.03, 0.04])

# 5 yr at 2% then 2.5 yr at 3%
print(round(ir.sn(7.5), 6))    # 1.188760
# Duality: sn(n) * vn(n) == 1
print(round(ir.sn(7.5) * ir.vn(7.5), 12))   # 1.0
```

## Calculation mode (`Config` is the source of truth)

`LifeTable` actuarial methods and `InterestRate.a()` / `InterestRate.ä()` both read
the global calculation mode from `Config` (or the `config` alias).  There is **no**
`calculation_mode` parameter on the public `InterestRate(...)` constructor, and
`LifeTable` exposes no per-instance override — set the mode once for the process:

```python
from lactuca import Config, InterestRate, LifeTable

Config().calculation_mode = "discrete_precision"   # default
ir = InterestRate(0.03)
print(ir.calculation_mode)   # mirrors Config

lt = LifeTable("PASEM2020_Rel_1o", "m")
print(lt.calculation_mode)   # same global setting
```

The read-only `InterestRate.calculation_mode` property reflects `Config` unless an
internal metadata override is present (used for serialization, not typical user
configuration).  To change how annuities are valued, update
`Config().calculation_mode` before calling table or `InterestRate` methods — see
{doc}`calculation_modes` and {doc}`configuration`.

## Using with table methods

Pass `ir` to any `LifeTable` method via the `ir` keyword argument.  A plain float is also accepted
and will be wrapped automatically:

```python
from lactuca import InterestRate, LifeTable

lt = LifeTable("PASEM2020_Rel_1o", "m")
ir = InterestRate(0.03)

print(round(lt.ax(65, ir=ir), 4))     # explicit InterestRate object
print(round(lt.ax(65, ir=0.03), 4))   # plain float shorthand (equivalent)
```

**Heterogeneous curves in portfolio batch:** when each policy needs a different term
structure or constant rate, pass a per-policy `ir` array to `LifeTable` batch methods
(not to `InterestRate.a()` batch, which shares one curve per instance):

```python
from lactuca import InterestRate, LifeTable

lt = LifeTable("PASEM2020_Rel_1o", "m")
ages = [55, 60, 65, 70]
ir_flat = InterestRate(0.03)
ir_curve = InterestRate(terms=[5, 5], rates=[0.02, 0.03, 0.04])

# One flat rate and one piecewise curve in the same portfolio batch
result = lt.ax(ages, n=20, ir=[ir_flat, ir_flat, ir_curve, ir_curve])
```

### Setting a default rate on the table

`LifeTable.interest_rate` is a property that stores a default rate for the table instance.
Once set, every calculation method uses it automatically — no need to repeat `ir=` on
each call:

```python
from lactuca import InterestRate, LifeTable

lt = LifeTable("PASEM2020_Rel_1o", "m")
lt.interest_rate = 0.03          # set once; accepts float or InterestRate

print(round(lt.ax(65), 4))       # uses 0.03
print(round(lt.ax(65, ir=0.04), 4))  # overrides to 0.04 for this call only
print(round(lt.ax(65), 4))       # back to 0.03 (override was not permanent)
```

You can also pass the rate at construction time:

```python
from lactuca import InterestRate, LifeTable

lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)
print(round(lt.ax(65), 4))       # 0.03 is the default
```

`interest_rate` accepts a full `InterestRate` object — useful when you need a term structure
rather than a single flat rate:

```python
from lactuca import InterestRate, LifeTable

ir_curve = InterestRate(terms=[5, 5, 10], rates=[0.025, 0.03, 0.035, 0.04])
lt = LifeTable("PASEM2020_Rel_1o", "m")
lt.interest_rate = ir_curve

print(round(lt.ax(65), 4))       # discounted with the full term structure
```

Set `lt.interest_rate = None` to require explicit `ir=` on every call (useful in
multi-rate sensitivity analyses to avoid accidentally using a stale default).

## Nominal rates

Convert between nominal and effective rates:

```python
from lactuca import InterestRate

ir = InterestRate(0.03)

print(round(ir.i_m(12), 6))    # 0.029595  nominal rate convertible monthly i^(12)
print(round(ir.d_m(12), 6))    # 0.029522  nominal discount rate d^(12)

# Ordering holds for i > 0, m > 1:
#   d^(m) < δ < i^(m) < i
#   0.029522 < 0.029559 < 0.029595 < 0.03
assert ir.d_m(12) < ir.delta() < ir.i_m(12) < ir.get_rate(0.0)

# m=1 returns annual effective rates directly
print(round(ir.i_m(1), 6))    # 0.03       (== i, no conversion)
print(round(ir.d_m(1), 6))    # 0.029126   (== d = i/(1+i))
```

## Piecewise term structure — discount formula

For a term structure with segments of lengths $\Delta_1, \Delta_2, \ldots$
and corresponding annual effective rates $i_1, i_2, \ldots$, the discount factor
to time $t$ is the product of the discount factors for each segment spanned:

$$v(t) = \prod_{k=1}^{K} \left(\frac{1}{1+i_k}\right)^{\min(\Delta_k,\, t - T_{k-1}^+)}$$

where $T_{k-1}^+$ is the cumulative duration at the start of segment $k$.  Lactuca
evaluates this automatically: pass a piecewise `InterestRate` and the correct compounding
is handled for any time $t$.

## Calendar conventions and `days_per_year`

Lactuca uses `days_per_year` to convert between calendar time and actuarial "years".
This setting lives on the global `Config` singleton and affects all `InterestRate`
calculations that involve date-based durations.  The default value is `365.25`.
The allowed values are:

| Value | Convention | Typical use |
|-------|-----------|-------------|
| `360` | 30/360 bond convention | Historical bond calculations |
| `365` | Actual/365 | Calendar-day exact |
| `365.25` | Mean Gregorian year | **EIOPA risk-free curves**, IAA standard (**default**) |
| `365.2425` | Gregorian calendar average | High-precision geodetic |
| `366` | Leap year | Special-purpose only |

For Solvency II reserving and EIOPA risk-free rate curves, use `365.25` — this is the
mean Gregorian year adopted in the EIOPA technical specifications and is consistent with
IAA international actuarial practice.  Using a wrong `days_per_year` introduces a
systematic reserve error proportional to the interest rate level.

Change the global setting via the `Config` singleton or the `config` alias:

```python
from lactuca import Config

Config().days_per_year = 365.25   # EIOPA / IAA standard (this is also the default)
```

Or equivalently with the lowercase alias:

```python
from lactuca import config

config.days_per_year = 365.25
```

:::{note}
Both `Config()` and `config` refer to the same process-global singleton.  A change
made via one is immediately visible through the other.  See {doc}`configuration` for
the full list of configurable settings and persistence options.
:::

## EIOPA risk-free rate curve (Solvency II)

The EIOPA risk-free rates are published as spot rates per maturity.  To use them in
Lactuca, convert to the required piecewise format (segment lengths + forward rates):

```python
from lactuca import InterestRate, config

config.days_per_year = 365.25   # required for EIOPA curves

# Example: EIOPA EUR RFR curve (illustrative — use actual published rates)
# Maturities 1, 5, 10, 20, 30 years; len(terms) + 1 == len(rates)
# Equivalent annual effective forward rates per segment:
ir_eiopa = InterestRate(
    terms=[1, 4, 5, 10, 10],                                         # 5 segment lengths
    rates=[0.0320, 0.0340, 0.0355, 0.0345, 0.0330, 0.0320],          # 6 rates (last applies indefinitely past yr 30)
)
```

> **Tip:** If you have spot (zero-coupon) rates $s_t$ at maturities $t_1 < t_2 < \ldots$,
> the implied forward rate for segment $[t_{k-1}, t_k]$ is:
> $f_k = \left(\frac{(1+s_{t_k})^{t_k}}{(1+s_{t_{k-1}})^{t_{k-1}}}\right)^{1/(t_k - t_{k-1})} - 1$

**Batch / DataFrame usage:** `ir=` accepts a Pandas or Polars `Series` directly.
A numeric `Series` is converted to per-policy floats.  An **object-dtype** `Series` of
`InterestRate` instances (including piecewise curves) is treated identically to a list —
no `.to_numpy()` is needed.  Mixing `InterestRate` objects and numeric values in a single
`Series` raises `ValueError`.
See {doc}`batch_calculations` — *Broadcasting rules* for the complete parameter table.

## Curve diagnostics (`curve_analysis` / `validate`)

`InterestRate.curve_analysis()` and `InterestRate.validate()` return a `curve_policy`
dict describing **what the discount engine supports** — not regulatory compliance:

| Flag | Meaning |
|------|---------|
| `negative_rates_allowed` | Zero and negative segment rates are accepted |
| `flat_or_piecewise_discount` | Constant or piecewise-flat term structures |
| `multi_scenario_curves` | Named scenario container present |

```python
from lactuca import InterestRate

ir = InterestRate(terms=[5, 5], rates=[0.02, 0.03, 0.04])
summary = ir.curve_analysis()
print(summary["curve_policy"])
# {'negative_rates_allowed': True, 'flat_or_piecewise_discount': True,
#  'multi_scenario_curves': False}
```

:::{warning}
`curve_policy` is **not** a Solvency II or IFRS 17 sign-off.  Lactuca discounts
cash flows with user-supplied curves; it does not load official EIOPA RFR files,
compute Risk Adjustment, CSM, or SCR modules.  Build BEL inputs manually from
published curves when required.
:::

## See also

- {doc}`growth_conventions` — `GrowthRate` for benefit escalation
- {doc}`commutation_functions` — how `ir` is used in Dx, Nx, etc.
- {doc}`calculation_modes` — how interest rate interacts with calculation mode

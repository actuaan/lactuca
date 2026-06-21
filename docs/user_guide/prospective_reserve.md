# Prospective Reserves and the `ts` Parameter

The **prospective reserve** ${}_{t}V$ at elapsed time $t$ for a contract issued to a life
aged $x$ is the difference between the actuarial present value of all future benefits and
the actuarial present value of all future net premiums, both evaluated from the current
attained age $x + t$:

$$
{}_{t}V_x = \text{APV}(\text{future benefits at age }x+t)
          - \text{APV}(\text{future premiums at age }x+t)
$$

In Lactuca, both present values are computed by calling the same `äx`, `ax`, `Ax` (or
joint-life equivalents) with the **effective age** $x + t$.  The `ts` parameter is the
mechanism that shifts the calculation to that age without requiring the caller to
recompute $x + t$ and adjust the remaining term manually.

The `ts` parameter (`ts` is short for *time shift*) represents the number of years
elapsed since the reference age `x`.  Two equivalent parameterisations are equally valid:

- **x = inception age, ts = total elapsed time** (years since the contract start).
- **x = age at the last elapsed whole-year anniversary, ts = fractional remainder**
  of the current policy year.

Both produce identical results because the engine only uses $x + ts$ internally.
It is available on all main life-contingency methods: `äx`, `ax`, `Ax` and their
joint-life variants (`äxy`, `äxyz`, `Axy`, `Axyz`), as well as the pure endowment
`nEx`.

`ts` enables **prospective reserve calculations** at any point in the life of a
contract:

- **Integer `ts`** (e.g. `ts=3`): the valuation falls exactly on a policy
  anniversary.  This is the standard case for annual statutory reserves.
- **Fractional `ts`** (e.g. `ts=3.5`): the valuation falls between two anniversaries.
  The survival and discounting calculations are mathematically exact.  When a
  `GrowthRate` is also provided, Lactuca applies the actuarial convention that benefit
  revaluations occur at whole-year anniversaries; a `UserWarning` is emitted so the
  user can review that convention before accepting the result (see
  [Growth rates and `ts`](#growth-rates-and-ts)).

:::{note}
**`ts` vs `t` — two orthogonal concepts**

In actuarial notation the symbol $t$ typically denotes a **future duration** — the
time span measured forward from a given age, as in the survival probability
${}_{t}p_x = l_{x+t}/l_x$.  The parameter *name* `ts` in Lactuca is deliberately
distinct: it represents **elapsed time** measured backward, from the reference point
to the current valuation date.  The engine converts `ts` into an age shift and a
duration reduction **before** any survival calculation is performed.  Once the
effective age and duration are established, the internal survival functions use `t`
(duration) in the classical sense.
:::

---

## How `ts` transforms the calculation

The caller always passes the **reference age** `x` and the **full remaining term**
`n` measured from that reference.  The engine applies four internal transformations:

$$x_{\text{eff}} = x + ts$$

$$d_{\text{eff}} = \max(d - ts,\; 0) \qquad ts_{\text{eff}} = \max(ts - d,\; 0)$$

$$n_{\text{eff}} = n - ts_{\text{eff}}$$

The shift first **absorbs any deferment**: if $ts \le d$, the deferment decreases and
the duration is unchanged ($ts_{\text{eff}} = 0$, $n_{\text{eff}} = n$).  Only the
portion of $ts$ that *exceeds* $d$ shortens the remaining duration.  In the common
case $d = 0$ this simplifies to $n_{\text{eff}} = n - ts$.

:::{note}
**Pure endowments** (`nEx`, `nExy`, `nExyz`, `nEjoint`) do not accept a deferment
parameter, so the formula simplifies to $x_{\text{eff}} = x + ts$ and
$n_{\text{eff}} = \max(n - ts, 0)$.
:::

The call signature is therefore:

```python
lt.äx(x, n=n, ts=ts, ir=ir)          # d=0 → n_eff = n - ts
lt.äx(x, n=n, d=d, ts=ts, ir=ir)     # general case
```

When `ts=0` (the default) no shift is applied; the result is the standard actuarial
present value at age `x` — the initial cost of the contract.

:::{note}
If $ts \ge d + n$ the contract has fully elapsed and the present value is **zero**.
Lactuca returns `0.0` immediately in that case without raising an error.  Passing a
negative `ts` raises `ValueError`.

For **whole-life calculations** (`n=None`) the effective duration is computed as
$n_{\text{eff}} = \omega - x_{\text{eff}} - d_{\text{eff}}$, where $\omega$ is the
terminal age of the table.  The shift shortens the residual lifetime automatically;
no finite `n` is required.
:::

---

## Basic usage

The most common case is an **integer `ts`**: the valuation falls on a future policy
anniversary.  The calculation reduces to a standard annuity starting from the attained
age $x + ts$:

```python
from lactuca import LifeTable, config

config.decimals.annuities = 4
lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)

# Standard anniversary reserve (ts = 0, no shift)
reserve_ann = lt.äx(50, n=15)
print(reserve_ann)

# Reserve at the 3rd anniversary: ts = 3 advances age to 53, remaining term to 12
reserve_3 = lt.äx(50, n=15, ts=3)
print(reserve_3)

config.reset()
```

**Fractional `ts`** is used when the valuation date falls between two anniversaries.
The survival and discounting are computed exactly for the fractional age:

```python
from lactuca import LifeTable, config

config.decimals.annuities = 4
lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)

# Mid-year valuation: 0.5 years since last anniversary
reserve_mid = lt.äx(50, n=15, ts=0.5)
print(reserve_mid)

# Quarter-year valuation: 0.25 years since last anniversary
reserve_off = lt.äx(50, n=10, ts=0.25)
print(reserve_off)

# 2.75 years since inception: 2 full anniversaries elapsed, 0.75 yr into the third year
reserve_multi = lt.äx(50, n=10, ts=2.75)
print(reserve_multi)

config.reset()
```

---

## `ts` vs `d` — key distinction

:::{seealso}
For a standalone introduction to the `d` parameter — concept, formulas, code examples,
and joint-life products — see {doc}`deferment`.
:::

`ts` and `d` look similar but serve different roles:

| Parameter | Meaning | Affects |
|-----------|---------|---------|
| `ts` | Time *already elapsed* since last anniversary | Starting age, payment-grid origin, growth schedule |
| `d` | Future *deferral* before payments begin | First payment is at $t = d_{\text{eff}}$ after current valuation age |

The crucial algebraic interaction: $ts$ **consumes deferment first**.  The deferment
that remains after the shift is $d_{\text{eff}} = \max(d - ts, 0)$, and only the
portion of $ts$ beyond $d$ reduces the term.

| $ts$ vs $d$ | $d_{\text{eff}}$ | $n_{\text{eff}}$ |
|---|---|---|
| $ts < d$ (shift is within deferment) | $d - ts$ | $n$ (unchanged) |
| $ts = d$ (shift exactly exhausts deferment) | $0$ | $n$ (unchanged) |
| $ts > d$ (shift extends past deferment) | $0$ | $n - (ts - d)$ |

Example — a 5-year-deferred, 20-year annuity evaluated at `ts=0.5` (mid-year):

```python
from lactuca import LifeTable, config

config.decimals.annuities = 4
lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)

# ts=0.5 < d=5 → deferment is only partially consumed: d_eff = 4.5, n_eff = 20
reserve_combined = lt.äx(50, n=20, d=5, ts=0.5)
print(reserve_combined)

# ts=6 > d=5 → deferment fully consumed: d_eff = 0, n_eff = 20 - (6-5) = 19
reserve_past_d = lt.äx(50, n=20, d=5, ts=6)
print(reserve_past_d)

config.reset()
```

:::{note}
**`ts` with explicit `cashflow_times`**

For methods that accept an explicit payment schedule (`cashflow_times`), `ts` still
applies but with two differences:

1. Each payment time is shifted relative to the valuation date:
   payments at times $< ts_{\text{eff}}$ (in the past) are **silently dropped**.
2. Integer enforcement (`config.force_integer_ts`) is bypassed — when the caller
   controls the exact payment grid, fractional shifts never raise or warn.
:::

---

## Integer-only mode (`force_integer_ts`)

By default, Lactuca accepts any non-negative float as `ts`.  Without a `GrowthRate`,
fractional `ts` is mathematically exact: survival probabilities and discount factors
are computed at the precise fractional attained age.  When a `GrowthRate` *is* active,
the growth schedule is advanced by `int(ts)` complete anniversaries (see
[Growth rates and `ts`](#growth-rates-and-ts) below).  For portfolios where only
anniversary-date reserves are required (e.g. end-of-year statutory convention), set
`config.force_integer_ts = True` to make Lactuca raise a `ValueError` on any
fractional `ts`:

```python
from lactuca import LifeTable, config

config.force_integer_ts = True          # fractional ts now raises ValueError
config.decimals.annuities = 4

lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)
lt.äx(50, n=15, ts=0.5)
# ValueError: [äx] Parameter 'ts' (shift) must be an integer value (got ts=0.5).
# To allow fractional shifts, set 'Config.force_integer_ts = False'.

config.reset()                          # restore defaults after the example above
```

The following table summarises all input combinations:

| `ts` value | `force_integer_ts` | `GrowthRate` present | Behaviour |
|---|---|---|---|
| Integer | `False` or `True` | Any | Accepted silently |
| Fractional | `False` | No | Accepted silently |
| Fractional | `False` | Yes | `UserWarning` emitted (see [Growth rates and `ts`](#growth-rates-and-ts)) |
| Fractional | `True` | Any | `ValueError` raised |

:::{note}
`force_integer_ts` is enforced by **annuity and insurance** methods (`ax`, `äx`,
`Ax`, and all joint-life variants). Pure endowment methods (`nEx`, `nExy`, `nExyz`,
`nEjoint`) currently do not enforce this setting — they accept fractional `ts`
regardless of its value, because endowments never carry a `GrowthRate`.
:::

---

## Piecewise interest rate with `ts`

For non-constant `InterestRate` term structures, the engine automatically calls
`ir.shifted(ts)` to align the yield curve with the valuation date.  No user action
is needed:

```python
from lactuca import LifeTable, InterestRate, config

config.decimals.annuities = 4

# Piecewise curve: 2.5% for years 0–5, 3.0% for years 5–10, 3.5% thereafter
# 2 terms → 3 rates required (last rate applies indefinitely)
ir = InterestRate(terms=[5, 5], rates=[0.025, 0.030, 0.035])
lt = LifeTable("PASEM2020_Rel_1o", "m")

# The yield curve is automatically shifted by ts=2.5 yr internally:
# effective curve from valuation date = 2.5% for 2.5 yr, 3.0% for 5 yr, 3.5% thereafter
reserve = lt.äx(50, n=10, ts=2.5, ir=ir)
print(reserve)

config.reset()
```

:::{note}
`ir.shifted(ts)` is only invoked when `ts > 0` and the interest rate is non-constant.
A flat rate is translation-invariant and is never shifted.
:::

---

(growth-rates-and-ts)=
## Growth rates and `ts`

When a `GrowthRate` schedule is combined with `ts`, the engine calls `gr.shifted(ts)`
internally.  Benefit revaluations follow the actuarial convention of whole-year policy
anniversaries: `shifted()` advances the schedule by `int(ts)` completed anniversaries,
not by the raw `ts` value.  This is correct and expected for **integer `ts`**:

```python
from lactuca import LifeTable, GrowthRate, config

config.decimals.annuities = 4
lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)

# Growth schedule: 3% for first 2 years, then 2% indefinitely
gr = GrowthRate(rates=[0.03, 0.02], terms=[2])

# ts=2: exactly 2 anniversaries elapsed → gr.shifted(2) consumes the first segment
# → remaining schedule from valuation date: 2% (constant second segment)
reserve = lt.äx(50, n=8, ts=2, gr=gr)
print(reserve)

config.reset()
```

With **fractional `ts`** and a `GrowthRate`, the same integer-anniversary convention
applies: `ts=2.5` advances the schedule by `int(2.5) = 2` anniversaries, not 2.5.
Because this truncation may be unexpected, Lactuca emits a `UserWarning` to prompt
the user to confirm the convention is appropriate for their calculation:

```python
from lactuca import LifeTable, GrowthRate, config

config.decimals.annuities = 4
lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)
gr = GrowthRate(rate=0.02)  # constant 2% growth schedule

reserve = lt.äx(50, n=15, ts=0.5, gr=gr)
# UserWarning: [äx] Fractional 'ts' (shift) detected (got ts=0.5).
#   GrowthRate schedules use whole-year anniversary indices, consistent with
#   actuarial convention.  Set 'Config.force_integer_ts = True' to reject
#   fractional shifts.
print(reserve)

config.reset()
```

:::{note}
For fractional `ts` with a `GrowthRate`:

- **Survival and discounting** are computed exactly at the fractional attained age.
- **Growth indexing** follows the convention documented in {doc}`growth_conventions`:
  the schedule is advanced by `int(ts)` complete anniversaries.  The growth factor
  in force during the fractional year before the next anniversary is that of the last
  whole-year milestone — consistent with the actuarial standard that benefit
  revaluations (CPI indexation, salary scales) operate on complete policy years.

To require strictly integer `ts` and bypass this convention entirely, set
`config.force_integer_ts = True`.
:::

For further detail on growth shifts see {doc}`growth_conventions`.

---

## Mode compatibility

| `calculation_mode` | Integer `ts` | Fractional `ts` |
|--------------------|:---:|:---:|
| `discrete_precision` | ✅ | ✅ |
| `discrete_simplified` | ✅ | ✅ |
| `continuous_precision` | ✅ | ✅ |
| `continuous_simplified` | ✅ | ✅ |

All four modes support fractional `ts`.  For highest accuracy at fractional ages use
`discrete_precision` (exact $\ell_x$ interpolation under the Uniform Distribution of Deaths
(UDD) assumption, `config.lx_interpolation = "linear"`) or `continuous_precision` (numerical
integration over the raw $\ell_x$ curve under the Constant Force of Mortality (CFM)
assumption, `config.lx_interpolation = "exponential"`).

---

## Portfolio reserves

Compute reserves for multiple policies by iterating over a portfolio.  For each
policy pass the reference age, the full remaining term from that reference, and the
elapsed fraction — the engine applies the shift internally.  Each policy requires a
separate call, so a plain list comprehension is the natural approach:

```python
from lactuca import LifeTable, config

config.decimals.annuities = 4
lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)

# Reference age (last anniversary), full remaining term, elapsed fraction this year
ages   = [50, 55, 60, 65]
terms  = [20, 15, 10,  5]
shifts = [0.0, 0.25, 0.5, 0.75]

reserves = [
    lt.äx(age, n=term, ts=ts_val)
    for age, term, ts_val in zip(ages, terms, shifts)
]
print(reserves)

config.reset()
```

---

## Net premium and prospective reserve — worked example

The prospective reserve formula connects directly to the **net level premium** — the
constant annual benefit payment $P$ chosen so that the contract is actuarially fair at
issue (${}_{0}V = 0$).  For an $n$-year term insurance:

$$
{}_{0}V = A^1_{x:\overline{n}|} - P \cdot \ddot{a}_{x:\overline{n}|} = 0
\quad \Longrightarrow \quad
\boxed{P = \frac{A^1_{x:\overline{n}|}}{\ddot{a}_{x:\overline{n}|}}}
$$

Once $P$ is known, the **prospective reserve** at any elapsed duration $t$ is:

$$
{}_{t}V = A^1_{x+t:\overline{n-t}|} - P \cdot \ddot{a}_{x+t:\overline{n-t}|}
$$

where $A^1_{x+t:\overline{n-t}|}$ and $\ddot{a}_{x+t:\overline{n-t}|}$ are the
actuarial present values of the future benefit and the premium annuity-due for the
remaining term $n - t$, evaluated at the attained age $x + t$.  In Lactuca, both are
obtained from the same table instance via the `ts=t` parameter:

```python
from lactuca import LifeTable, config

config.decimals.annuities = 6
config.decimals.insurances = 6

lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)

x = 40    # issue age (integer years)
n = 25    # term in years

# ── Step 1: compute net level premium at issue (ts = 0) ──────────────────────
A0   = lt.Ax(x, n=n)      # APV of term insurance benefit at inception
a0   = lt.äx(x, n=n)      # APV of premium annuity-due at inception
P    = A0 / a0             # net level premium per unit of sum insured
print(f"Net level premium P = {P:.6f}")

# ── Step 2: prospective reserve at each anniversary ──────────────────────────
print(f"\n{'t':>4}  {'A(x+t)':>12}  {'äx(x+t)':>12}  {'tV':>12}")
for t in range(0, n + 1, 5):
    At  = lt.Ax(x, n=n, ts=t)    # APV of benefits from attained age x+t
    at  = lt.äx(x, n=n, ts=t)    # APV of premiums from attained age x+t
    tV  = At - P * at             # prospective reserve
    print(f"{t:>4}  {At:>12.6f}  {at:>12.6f}  {tV:>12.6f}")

config.reset()
```

:::{note}
The reserve at `ts=25` (end of term) is zero because the contract has fully elapsed:
both `At` and `at` equal zero, so `tV = 0`.  The reserve starts at zero at issue,
rises through mid-term, and returns to zero at expiry — the typical shape for a term
policy under net premium valuation.
:::

### Whole-life variant

For a whole-life insurance (`n=None`) the same pattern applies; the premium annuity
runs for the rest of life:

$$
P_x^{\infty} = \frac{A_x}{\ddot{a}_x}
\qquad
{}_{t}V_x^{\infty} = A_{x+t} - P_x^{\infty} \cdot \ddot{a}_{x+t}
$$

```python
from lactuca import LifeTable, config

config.decimals.annuities = 6
config.decimals.insurances = 6

lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)

x = 40

# Net level premium for a whole-life policy
A_wl  = lt.Ax(x)       # whole-life insurance at issue
a_wl  = lt.äx(x)       # whole-life annuity-due at issue
P_wl  = A_wl / a_wl
print(f"Whole-life premium P = {P_wl:.6f}")

# Reserve at selected policy years
for t in [0, 5, 10, 20, 30]:
    tV = lt.Ax(x, ts=t) - P_wl * lt.äx(x, ts=t)
    print(f"  t={t:2d}: tV = {tV:.6f}")

config.reset()
```

:::{note}
For a whole-life insurance, ${}_{t}V$ increases monotonically and approaches 1 as
$t \to \omega - x$.  This is the classic "savings" accumulation in whole-life
contracts.  The reserve is identically zero at $t = 0$ (the actuarial equivalence
principle), which is a useful numerical self-check: verify that
`lt.Ax(x) - P_wl * lt.äx(x)` returns a value indistinguishable from zero before
proceeding to production calculations.
:::

---

## Methods that support `ts`

`ts` is available on all **annuity**, **insurance**, and **pure-endowment** methods:

| Category | Methods |
|----------|---------|
| Annuities (single-life) | `ax`, `äx` |
| Annuities (joint-life) | `axy`, `äxy`, `axyz`, `äxyz`, `ajoint`, `äjoint` |
| Insurances | `Ax`, `Axy`, `Axyz`, `Afirst` |
| Pure endowments | `nEx`, `nExy`, `nExyz`, `nEjoint` |

**Probability, commutation, and life-expectancy methods do not accept `ts`** and
always operate at the age passed directly.  To evaluate the $t$-year survival
probability at the attained age after a shift, pass `x + ts` explicitly and use `t`
as the forward duration:

```python
# t-year survival from the shifted attained age (t here is a duration, not ts)
survival = lt.tpx(x + ts, t=n)
```

Here `t=n` is the **future duration** (years to survive), wholly distinct from `ts`
(years already elapsed).

---

## See also

- {doc}`life_annuities_guide` — `äx`, `ax`: the premium annuity in $P = A_x / \ddot{a}_x$
- {doc}`life_insurances_guide` — `Ax`, `nEx`: the insurance numerator in the premium equation
- {doc}`growth_conventions` — how `GrowthRate.shifted()` aligns benefit revaluation with anniversaries
- {doc}`inspecting_cashflows` — examine the cash-flow arrays underlying the reserve
- {doc}`calculation_modes` — precision trade-offs for fractional-age calculations
- {doc}`batch_calculations` — compute $_tV$ for a full portfolio in one vectorized call (building block for BEL / IFRS 17 FCF — not full GMM, RA, or CSM; see {ref}`regulatory-reporting-scope` in {doc}`calculation_modes`)
- {doc}`../formulas` — formal prospective reserve formula

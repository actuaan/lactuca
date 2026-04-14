(errors-reference)=
# Error Reference

This page is the definitive reference for `ValueError` and `TypeError` exceptions raised
by Lactuca's public API.  For each condition you will find the exact **message pattern**,
the **cause**, a concrete **fix**, and **code examples** showing the triggering call
alongside the correct alternative.

:::{important}
**Scope of this reference** — this page covers only errors that are *non-obvious*,
*cross-cutting* (affecting multiple methods or components), or that arise from
*configuration-of-state surprises* (e.g. restrictions that only appear after the object
has been built in a particular way).  Simple parameter validation whose error message is
self-explanatory (e.g. *"n must be non-negative"*) is intentionally omitted.
:::

:::{note}
Error messages include dynamic context — the calling method name and the actual input
values — so the exact error message text will differ from the patterns shown here.
The **message patterns** below use `{placeholders}` for those dynamic parts.
:::

## Quick reference

Scan for your exception type or a keyword from the error message, then follow the link for
the full explanation, cause, and fix.

| Category | Exception | Error condition | Trigger |
|---|---|---|---|
| **Age & term** | `ValueError` | [Integer age required](#integer-age-required) | Fractional age passed to `Dx`, `Nx`, `Cx`, `Mx`, `Sx`, `Rx`, `Lx`, `Tx`, or `ex` |
| **Age & term** | `ValueError` | [Fractional age required](#fractional-age-required) | Integer age passed to `ex_continuous`, `Lx_continuous`, or `Tx_continuous` |
| **Age & term** | `ValueError` | [Age out of range](#age-out-of-range) | Age $< 0$ passed to any method, or age $> \omega$ passed to commutation/annuity/insurance methods (`lx` returns 0.0 for ages $> \omega$) |
| **Age & term** | `ValueError` | [Duration, deferment, or time-shift out of range](#term-must-be-positive) | `n` or `ts` negative or non-finite; `d` negative only |
| **Interest rate** | `ValueError` | [Rate ≤ −1](#rate--1) | Force of interest (`delta(t)`) or discount factor (`vn(n)`) computed with rate $\leq -1$ |
| **Interest rate** | `ValueError` | [Piecewise rate inputs](#piecewise-rate-input-errors) | Wrong number of rates or non-positive terms |
| **Interest rate** | `TypeError` | [Piecewise rate inputs](#piecewise-rate-input-errors) | Non-finite (NaN or Inf) value in the `rates=` array |
| **Table data** | `ValueError` | [Terminal age condition](#omega-condition) | $q_\omega \neq 1.0$ in a user-supplied `.ltk` file |
| **Table data** | `ValueError` | [Generational table](#generational-table-errors) | Missing, unexpected, or out-of-range `cohort`; or missing `base_year` metadata |
| **Configuration** | `ValueError` | [Invalid setting value](#invalid-setting-value) | Disallowed value for a config setting |
| **Configuration** | `ValueError` | [Decimal places](#decimal-places) | Negative or non-integer value for `config.decimals.*` |
| **Input arrays** | `ValueError` | [Cashflow arrays](#mismatched-cashflow-arrays) | `cashflow_times` and `cashflow_amounts` differ in length |
| **Input arrays** | `ValueError` | [Broadcasting](#broadcasting-errors) | `x` and `x0` arrays passed to `InterestRate` have incompatible shapes |
| **Interest rate** | `TypeError` | [Wrong type for `interest_rate`](#wrong-type-interest-rate) | `int` or other non-`float` passed to the `interest_rate` property setter or constructor |
| **Cashflow utilities** | `ValueError` | [Invalid payment frequency](#invalid-payment-frequency) | Invalid `m` passed to `generate_payment_times` or `GrowthRate.amounts` |
| **Cashflow utilities** | `ValueError` | [Invalid selected periods](#invalid-selected-periods) | Period value outside `[1, m]` in `generate_payment_times` |
| **Cashflow utilities** | `TypeError` | [Invalid selected periods](#invalid-selected-periods) | Non-integer value in `selected_periods` |
| **Cashflow utilities** | `ValueError` | [Tier structure](#tier-structure-errors) | `len(values) ≠ len(breakpoints) + 1`, or breakpoints not strictly ascending |
| **Cashflow utilities** | `ValueError` | [Non-finite cashflow inputs](#non-finite-cashflow-inputs) | NaN or Inf in `times`, `breakpoints`, `values`, or `start` |
| **Cashflow utilities** | `TypeError` | [times is None](#times-is-none) | `None` passed as `times` to `GrowthRate.amounts` |
| **GrowthRate** | `ValueError` | [GrowthRate constructor errors](#growthrate-constructor-errors) | Invalid `growth_type`, `rate`, `rates`/`terms`, or scenario dict |
| **GrowthRate** | `TypeError` | [GrowthRate constructor errors](#growthrate-constructor-errors) | `apply_from_first` not `bool`; `rate` is `bool` or non-numeric |
| **GrowthRate** | `ValueError` | [GrowthRate property setters](#growthrate-property-setters) | `rates` or `terms` setter: multi-scenario, constant, wrong shape or length |
| **GrowthRate** | `ValueError` | [GrowthRate.add_scenario errors](#growthrate-add-scenario-errors) | `scenario` not a `GrowthRate`; nested multi-scenario |
| **GrowthRate** | `TypeError` | [GrowthRate.add_scenario errors](#growthrate-add-scenario-errors) | `scenario` is not a `GrowthRate` instance |
| **GrowthRate** | `TypeError` | [GrowthRate.factor errors](#growthrate-factor-errors) | `t` is `None` (raises `TypeError`, not `ValueError`) |
| **Interest rate** | `ValueError` | [InterestRate advanced constructor](#interestrate-advanced-constructor) | Invalid `term_unit` passed to `InterestRate()` |
| **Interest rate** | `TypeError` | [InterestRate.add_scenario errors](#interestrate-add-scenario-errors) | `scenario` not an `InterestRate` instance |
| **Interest rate** | `ValueError` | [InterestRate.add_scenario errors](#interestrate-add-scenario-errors) | Nested multi-scenario `InterestRate` |
| **Interest rate** | `ValueError` | [InterestRate.vx validation errors](#interestrate-vx-validation) | Negative `x` or `x0`, or `x < x0` |
| **Interest rate** | `TypeError` | [InterestRate.i_m / d_m errors](#interestrate-im-dm-errors) | `m` not a plain `int` or `t` not numeric |
| **Interest rate** | `ValueError` | [InterestRate.i_m / d_m errors](#interestrate-im-dm-errors) | `m ≤ 0` or `t < 0` |
| **Interest rate** | `TypeError` | [InterestRate.get_average_force errors](#interestrate-get-average-force-errors) | `n` not numeric |
| **Interest rate** | `ValueError` | [InterestRate.get_average_force errors](#interestrate-get-average-force-errors) | `n ≤ 0` or discount factor ≤ 0 |

:::{note}
Most rows in the Quick reference table raise `ValueError`.  Rows that raise `TypeError`
are: Non-finite `rates=` value (piecewise interest rate), Wrong type for `interest_rate`,
Invalid selected periods (non-integer value), times is None, GrowthRate constructor type
checks, GrowthRate.factor None, GrowthRate.add_scenario, InterestRate.add_scenario,
InterestRate.i_m/d_m, InterestRate.get_average_force.
:::

---

(age-and-term-errors)=
## Age and term errors

These errors are raised when an age, duration, deferment, or time-shift argument is
outside the valid domain for the requested operation.

(integer-age-required)=
### Integer age required

**Exception**: `ValueError`

**Message pattern**:
```
[{method}] Commutation functions require integer ages. Got non-integer age(s): {values}.
```

**Cause**: A commutation function (`Dx`, `Nx`, `Cx`, `Mx`, `Sx`, `Rx`, `Lx`, `Tx`) or
the life-expectancy method `ex` received a fractional age.  Commutation functions are
discrete quantities defined only at integer ages; `ex` uses integer-age commutation
columns internally and shares the same constraint.

:::{note}
The lowercase `lx` and `dx` methods do **not** raise this error — they accept fractional
ages and perform interpolation according to `config.lx_interpolation`.

`Lx` (capital L) is the commutation function
$L_x = \int_0^1 l_{x+t}\,\mathrm{d}t$ that requires an integer age, while `lx`
(lowercase) is the $l_x$ survival function that interpolates at fractional ages.
The same pairing applies to `Tx` (capital, integer-age) vs `Tx_continuous` (fractional).
:::

**Fix**: Pass an integer or a float whose fractional part is within float64 rounding
tolerance ($\leq 10^{-12}$).  The same restriction applies to `ex`.
For the complete list of commutation functions and their signatures, see
{doc}`user_guide/commutation_functions`.

```python
from lactuca import LifeTable
lt = LifeTable("PASEM2020_Rel_1o", "m")

lt.ex(x=65)                   # ✔  integer age
lt.ex(x=65.0)                 # ✔  integer-valued float (treated as 65)
lt.Dx(x=45, ir=0.03)          # ✔  commutation function with explicit ir=
lt.Dx(x=45, ir=0.03, x0=25)   # ✔  x0 is the reference (start) age for discounting
lt.ex(x=65.5)                 # ✘ → ValueError: non-integer age
lt.Nx(x=45.7, ir=0.03)        # ✘ → ValueError: non-integer age
```

(fractional-age-required)=
### Fractional age required

**Exception**: `ValueError`

**Message pattern** (method-specific — each method hard-codes its own name and its
discrete alternative):
```
[ex_continuous] Requires fractional (non-integer) ages. Got integer age(s): {values}.
Use ex() for integer ages instead.
```

`Lx_continuous` and `Tx_continuous` produce analogous messages, naming `Lx()` and
`Tx()` as their respective discrete alternatives.

**Cause**: A continuous-domain method (`ex_continuous`, `Lx_continuous`,
`Tx_continuous`) received an integer or integer-valued float.  These methods compute
values via numerical integration for $x \notin \mathbb{Z}$.

**Fix**: Pass a fractional age, or use the corresponding discrete method for integer
ages: `ex(x)`, `Lx(x)`, or `Tx(x)` respectively.

```python
from lactuca import LifeTable
lt = LifeTable("PASEM2020_Rel_1o", "m")

lt.ex_continuous(65.5)         # ✔  fractional age
lt.ex_continuous(65.25)        # ✔  any non-integer age
lt.Lx_continuous(65.5)         # ✔  continuous version of Lx, same fractional-age rule
lt.ex_continuous(65.5, m=12)   # ✔  explicit m= (sub-intervals for numerical integration)
lt.ex_continuous(65.5, m=4)    # ✔  coarser integration (faster, less precise)
lt.ex(65)                      # ✔  use ex() at integer ages
lt.Lx(x=65)                   # ✔  use Lx() at integer ages
lt.ex_continuous(65)           # ✘ → ValueError: integer age
lt.ex_continuous(65.0)         # ✘ → ValueError: integer-valued float
lt.Lx_continuous(65)           # ✘ → ValueError: same rule (Lx_continuous)
```

(age-out-of-range)=
### Age out of range

**Exception**: `ValueError`

**Message pattern**: `"All ages x must be in [0, {omega}]"`

**Cause**: The age provided is negative or exceeds the table's terminal age $\omega$.
Most public methods that accept an age argument apply this check, including commutation
functions, annuity and insurance methods, and continuous methods
(`ex_continuous`, `Lx_continuous`, `Tx_continuous`).

:::{note}
`lx` is an exception: by actuarial convention, ages beyond $\omega$ are valid and return
`0.0` (all survivors have died).  Only **negative** ages raise `ValueError` for `lx`.
:::

**Fix**: Ensure the age satisfies $0 \leq x \leq \omega$.  Use `lt.omega` to read the
terminal age of a loaded table before computing.  Vectorised calls apply the same bounds
check element-wise: a single out-of-range value in an array triggers the error for the
entire call.

```python
from lactuca import LifeTable
lt = LifeTable("PASEM2020_Rel_1o", "m")

print(lt.omega)           # inspect the terminal age of this table
lt.lx(x=65)              # ✔  within [0, omega]
lt.lx(x=999)             # ✔  returns 0.0 — ages beyond omega follow actuarial convention
lt.Dx(x=50, ir=0.03)     # ✔  within [0, omega]
lt.Dx(x=999, ir=0.03)    # ✘ → ValueError: age > omega
lt.lx(x=-1)              # ✘ → ValueError: negative age
```

(term-must-be-positive)=
### Duration, deferment, or time-shift out of range

**Exception**: `ValueError`

| Parameter | Meaning | Condition | Message pattern |
|---|---|---|---|
| `n` | term in years | `n < 0` or `n` is NaN or Inf | `"[{method}] Parameter n (duration) must be finite and >= 0"` |
| `d` | deferment in years | `d < 0` | `"[{method}] Parameter d (deferment) must be >= 0"` |
| `ts` | time-shift in years | `ts < 0` or `ts` is NaN or Inf | `"[{method}] Parameter ts (shift) must be finite and >= 0"` |

:::{note}
`n = 0` does **not** raise — an annuity or insurance with zero term returns `0.0` without
an error.  `d = 0` and `ts = 0` are equally valid (no deferment, no time-shift).
For whole-life products, omit `n` entirely (it defaults to `None`).
:::

**Fix**: Pass `n >= 0`, `d >= 0`, and `ts >= 0`.  `n` and `ts` must also be finite
(NaN or Inf are rejected); `d` checks sign only.

```python
from lactuca import LifeTable
lt = LifeTable("PASEM2020_Rel_1o", "m")

lt.äx(65, n=20, ir=0.03)     # ✔  20-year temporary annuity
lt.äx(65, n=0, ir=0.03)      # ✔  zero-term returns 0.0, no error
lt.äx(65, ir=0.03)           # ✔  whole-life (n=None, the default)
lt.äx(65, n=-1, ir=0.03)              # ✘ → ValueError: n < 0
lt.äx(65, n=float('inf'), ir=0.03)    # ✘ → ValueError: n is non-finite
lt.äx(65, d=-2, ir=0.03)              # ✘ → ValueError: d < 0
lt.äx(65, ts=-0.5, ir=0.03)           # ✘ → ValueError: ts < 0
```

---

(interest-rate-errors)=
## Interest rate errors

These errors are raised by `InterestRate` — at construction (invalid rate values,
piecewise structure, or `term_unit`), through scenario management (`add_scenario`,
`active_scenario`), on method calls (`vx`, `i_m`, `d_m`, `get_average_force`), or when
an `ir=` argument to an annuity or insurance method carries an invalid rate — and by the
`LifeTable.interest_rate` property setter (and constructor argument) when it receives a
value of the wrong type.

(rate--1)=
### Rate ≤ −1

**Exception**: `ValueError`

**Message pattern**: depends on input type.

| Input type | Message pattern |
|---|---|
| Scalar rate | `"Rate {value} ≤ -1"` |
| Array of rates | `"Rates ≤ -1 at indices {indices}"` |

**Cause**: An annual effective rate $i \leq -1$ was used to compute either the
force of interest $\delta = \ln(1+i)$ or the discount factor $v^n = (1+i)^{-n}$.
Both operations share the same precondition: $(1+i) > 0$.  A rate of exactly $-1$
makes $(1+i) = 0$, so $\ln(1+i)$ is undefined and $(1+i)^{-n}$ is also undefined;
any $i < -1$ yields complex results for both expressions.
The check fires inside `InterestRate.delta(t)` and `InterestRate.vn(n)`,
but **not** at construction time.

**Fix**: Use $i > -1$.  For reference, typical life-annuity discount rates are in the
range $[0.01, 0.06]$, but any value $> -1$ is mathematically valid.

```python
from lactuca import InterestRate
InterestRate(0.03)              # ✔  3 % annual effective rate
InterestRate(-0.5)              # ✔  negative but > −1 (e.g. ECB negative deposit rate)
InterestRate(-1.0)              # ← construction succeeds; error fires on first use
InterestRate(-1.0).delta(1)    # ✘ → ValueError: rate ≤ −1 (ln(0) undefined)
InterestRate(-2.0).delta(1)    # ✘ → ValueError: rate < −1
InterestRate(-1.0).vn(5)       # ✘ → ValueError: rate ≤ −1 ((1+i) = 0)
InterestRate(-2.0).vn(1)       # ✘ → ValueError: rate < −1
```

:::{note}
The rate validation is **lazy**: `InterestRate(-1.0)` constructs without raising.
The `ValueError` fires when computing the force of interest via `InterestRate.delta(t)`
or the discount factor via `InterestRate.vn(n)` — both require $(1+i) > 0$
by the same actuarial precondition.  Rates in the range $(-1, 0)$ are economically
valid (e.g. the ECB maintained negative deposit rates of $-0.5~\%$ from 2019 to 2022)
and are accepted by both methods without raising.
:::

(piecewise-rate-input-errors)=
### Piecewise rate input errors

**Exception**: `ValueError` (or `TypeError` for non-finite rate values — see table)

**Message patterns**:

| Condition | Exception | Message pattern |
|---|---|---|
| `terms` or `rates` is empty | `ValueError` | `"terms and rates must not be empty"` |
| `len(rates) ≠ len(terms) + 1` | `ValueError` | `"rates must have length terms+1 (last rate applies indefinitely)"` |
| Any term is non-positive | `ValueError` | `"All terms must be positive"` |
| A rate is non-finite (NaN or Inf) | `TypeError` | `"InterestRate only accepts finite numeric values for 'rates', got {values}."` |

**Cause**: The piecewise `InterestRate` constructor validates `terms` and `rates` before
accepting them.  Each condition is checked in order; the first failing condition raises.

**Fix**: 
- `rates` must have exactly `len(terms) + 1` elements: one rate per interval plus one
  "tail" rate that applies indefinitely beyond the last term breakpoint.
- All `terms` must be strictly positive (no zero or negative durations).
- All rates must be real finite numbers — `math.nan`, `math.inf`, and `-math.inf` are rejected.
- All rates must also be economically valid ($> -1$); see [Rate ≤ −1](#rate--1).

```python
import math
from lactuca import InterestRate

# 2 terms → rates must have exactly 3 elements (len(terms) + 1):
ir = InterestRate(terms=[5, 10], rates=[0.03, 0.04, 0.05])  # ✔

# ✘ empty arrays — constructor rejects empty terms or rates:
InterestRate(terms=[], rates=[])                             # ✘ → ValueError

# ✘ only 2 rates for 2 terms — must be len(terms)+1 = 3:
InterestRate(terms=[5, 10], rates=[0.03, 0.04])             # ✘ → ValueError

# ✘ negative term duration:
InterestRate(terms=[-5, 10], rates=[0.03, 0.04, 0.05])      # ✘ → ValueError

# ✘ NaN rate — not finite (raises TypeError, not ValueError):
InterestRate(terms=[10], rates=[0.03, math.nan])             # ✘ → TypeError
```

(interestrate-advanced-constructor)=
### InterestRate advanced constructor

**Exception**: `ValueError`

**Message pattern**: `"Invalid term_unit: {value}. Must be one of {VALID_TERM_UNITS}"`

**Cause**: The `term_unit` argument controls how the `terms` sequence is interpreted.
Passing a string outside the four allowed values raises immediately.

**Allowed values**: `'years'`, `'months'`, `'weeks'`, `'days'` (default: `'years'`).

```python
from lactuca import InterestRate

InterestRate(0.03)                                    # ✔  default years
InterestRate(0.03, term_unit='months')                # ✔
InterestRate(terms=[6, 12], rates=[0.01, 0.012, 0.013], term_unit='months')  # ✔
InterestRate(0.03, term_unit='biweekly')              # ✘ → ValueError
InterestRate(0.03, term_unit='annual')                # ✘ → ValueError
```

(interestrate-add-scenario-errors)=
### InterestRate.add_scenario errors

**Exception**: `TypeError` or `ValueError`

**Message patterns**:

| Condition | Exception | Message pattern |
|---|---|---|
| `scenario` not `InterestRate` | `TypeError` | `"scenario must be an InterestRate instance, got {type}"` |
| Nested multi-scenario | `ValueError` | `"Nested multi-scenario InterestRate is not allowed. Add only simple scenarios (constant or piecewise)."` |

**Cause**: Each scenario must be a simple (constant or piecewise) `InterestRate`.  Passing
a multi-scenario container or any other type raises.

```python
from lactuca import InterestRate

ir = InterestRate({'base': 0.03})
ir.add_scenario('stress', InterestRate(0.05))               # ✔
ir.add_scenario('nested', InterestRate({'a': 0.03}))        # ✘ → ValueError: nested
ir.add_scenario('bad', 0.05)                                # ✘ → TypeError: not InterestRate
```

:::{note}
Setting `ir.active_scenario = 'unknown'` raises `ValueError: "Scenario 'unknown' not found."`
if the name does not exist. Use `ir.scenario_names` to list valid names.
:::

(interestrate-vx-validation)=
### InterestRate.vx validation errors

**Exception**: `ValueError`

**Message patterns**:

| Condition | Message pattern |
|---|---|
| `x` has negative values | `"'x' contains negative values."` |
| `x0` has negative values | `"'x0' contains negative values."` |
| `x < x0` element-wise | `"All values in 'x' must be greater than or equal to 'x0'."` |

**Cause**: `vx(x, x0)` computes the discount factor $v^{x - x_0}$ for a duration of
$x - x_0$ years.  Both arguments represent ages or time points and must be non-negative,
and `x` must be at least as large as `x0` (the reference start).

**Fix**: Ensure `x >= x0 >= 0` for all elements.

```python
import numpy as np
from lactuca import InterestRate

ir = InterestRate(0.03)
ir.vx(5)                                  # ✔  x0=0 by default
ir.vx(np.array([30, 40]), x0=25)          # ✔
ir.vx(-1)                                 # ✘ → ValueError: negative x
ir.vx(5, x0=-1)                           # ✘ → ValueError: negative x0
ir.vx(np.array([10, 20]), x0=np.array([15, 5]))  # ✘ → ValueError: x[0] < x0[0]
```

(interestrate-im-dm-errors)=
### InterestRate.i_m / d_m errors

**Exception**: `TypeError` or `ValueError`

`i_m(m, t)` computes the nominal rate $i^{(m)}$ and `d_m(m, t)` computes
the nominal discount rate $d^{(m)}$. Both share identical validation.

**Message patterns**:

| Condition | Exception | Message pattern |
|---|---|---|
| `m` not plain `int` (or is `bool`) | `TypeError` | `"m must be a plain integer, got {type}"` |
| `m ≤ 0` | `ValueError` | `"m must be a positive integer, got {m}"` |
| `t` not numeric (or is `bool`) | `TypeError` | `"t must be numeric, got {type}"` |
| `t < 0` | `ValueError` | `"t must be non-negative, got {t}"` |

**Cause**: `m` is the payment frequency (must be a positive plain `int`); `t` is the
period in years (non-negative).

```python
from lactuca import InterestRate

ir = InterestRate(0.03)
ir.i_m(12, 0)          # ✔  i^(12) at t=0
ir.d_m(2, 0)           # ✔  d^(2) at t=0
ir.i_m(0, 0)           # ✘ → ValueError: m <= 0
ir.i_m(12.0, 0)        # ✘ → TypeError: float, not int
ir.i_m(True, 0)        # ✘ → TypeError: bool
ir.d_m(12, -1)         # ✘ → ValueError: t < 0
ir.d_m(12, 't')        # ✘ → TypeError: not numeric
```

(interestrate-get-average-force-errors)=
### InterestRate.get_average_force errors

**Exception**: `TypeError` or `ValueError`

**Message patterns**:

| Condition | Exception | Message pattern |
|---|---|---|
| `n` not numeric | `TypeError` | `"Duration n must be numeric"` |
| `n ≤ 0` | `ValueError` | `"Duration n must be positive per actuarial practice, got {n}"` |
| Discount factor ≤ 0 | `ValueError` | `"Cannot calculate average force: discount factor {v} ≤ 0 results in undefined average force per actuarial standards"` |

**Cause**: The average force of mortality $\bar{\delta}_n = -\ln(v^n)/n$ requires
$n > 0$ and $v^n > 0$; the latter fails if the underlying rate is sufficiently
negative.

```python
from lactuca import InterestRate

ir = InterestRate(0.03)
ir.get_average_force(5)      # ✔
ir.get_average_force(0)      # ✘ → ValueError: n <= 0
ir.get_average_force(-1)     # ✘ → ValueError: n <= 0
ir.get_average_force('5')    # ✘ → TypeError
```

(wrong-type-interest-rate)=
### Wrong type for `interest_rate`

**Exception**: `TypeError`

**Message patterns**:

| Context | Message |
|---|---|
| `LifeTable.interest_rate` property | `"interest_rate must be a float, InterestRate, or None."` |
| `ir=` keyword argument | `"interest_rate must be a float, InterestRate, or None."` |

:::{note}
This `TypeError` applies to the **`interest_rate` property setter** and to
**`LifeTable.__init__`** (the `interest_rate=` constructor argument).  The `ir=` keyword
on annuity and insurance methods (`äx`, `ax`, `Ax`, etc.) is more permissive: it accepts
integers and automatically coerces them via `InterestRate(ir)` — so `lt.äx(65, ir=3)`
produces a 300 % discount rate without raising.  Always pass a `float` to avoid
unintentionally enormous discount rates.
:::

**Cause**: The `interest_rate` property setter (or the `interest_rate=` constructor
argument) received a value that is not a `float`, an `InterestRate` object, or `None`.
The most common mistake is passing an `int` such as `3` instead of the float `0.03`.

**Fix**: Use a `float` literal, an `InterestRate` object, or `None`.  Note that `3.0`
is a float but means 300 % — use `0.03` for 3%: `0.03`, `3.0 / 100`, or
`InterestRate(0.03)`.

```python
from lactuca import LifeTable, InterestRate
lt = LifeTable("PASEM2020_Rel_1o", "m")

# Property assignment:
lt.interest_rate = 0.03                 # ✔  float
lt.interest_rate = InterestRate(0.03)   # ✔  InterestRate object
lt.interest_rate = None                 # ✔  clears the table default
lt.interest_rate = 3                    # ✘ → TypeError  (int — use 0.03 for a 3 % rate)
lt.interest_rate = "0.03"               # ✘ → TypeError  (str not accepted)

# ir= keyword argument — more permissive than the property setter:
lt.äx(65, ir=0.03)                      # ✔  float
lt.äx(65, ir=InterestRate(0.03))        # ✔  InterestRate object
lt.äx(65, ir=3)                         # ✔  int coerced — 3 means 300 %, use 0.03 for 3 %
```

---

(table-construction-errors)=
## Table construction errors

These errors occur during table loading or construction — typically when a custom `.ltk`
file contains invalid data or when a generational table call omits required metadata.

(omega-condition)=
### Terminal age condition — $q_\omega \neq 1$

**Exception**: `ValueError`

**Message pattern**: `"Life table '{table}': qx_{sex}[omega={omega}] must be 1.0, got {value}."`

| `{sex}` value | Sex column |
|---|---|
| `m` | Male (`qx_m`) |
| `f` | Female (`qx_f`) |
| `u` | Unisex (`qx_u`) |

**Cause**: The annual probability of death at the terminal age $\omega$ must equal exactly
1.0 — all survivors must die by age $\omega$.  Deviations $> 10^{-4}$ from 1.0 are
rejected.

**Fix**: Set `qx[omega] = 1.0` in your source data.  Open the `.ltk` file and check the
last row of the relevant decrement column (`qx_m`, `qx_f`, or `qx_u`).  Values within
$10^{-4}$ of 1.0 (e.g. `0.9999998`) are silently clamped on load and will not raise;
only deviations $> 10^{-4}$ trigger this error.

:::{note}
All bundled Lactuca tables are pre-validated and will never raise this error.  This error
only occurs when loading a user-supplied `.ltk` file.  See
{doc}`user_guide/building_tables` for the `.ltk` format specification and the
`qx[omega] = 1.0` requirement.
:::

(generational-table-errors)=
### Generational table errors

**Exception**: `ValueError`

**Causes and message patterns**:

| Cause | Message pattern |
|---|---|
| `cohort` supplied to a static (non-generational) table | `"Cohort should not be specified for static tables."` |
| `cohort` omitted from a generational table call | `"Cohort must be set for generational tables."` |
| `base_year` absent from `.ltk` metadata | `"base_year metadata missing in table '{table}'."` |
| `cohort` value is not an integer | `"Cohort year must be an integer."` |
| `cohort` year outside valid range | `"Cohort year must be between 1900 and {year}."` (`{year}` = current calendar year) |

**Fix**:
- **Static table, `cohort` supplied**: remove the `cohort` argument from the `LifeTable(…)`
  call — static tables are indexed by attained age only.
- **Generational table, no `cohort`**: add `cohort=<birth_year>` (e.g. `cohort=1975`);
  the value is the policyholder's year of birth.
- **Missing `base_year`**: open the `.ltk` file and add `base_year = <year>` to the
  metadata section at the top of the file; see {doc}`user_guide/building_tables` for the
  full `.ltk` metadata format.
- **Non-integer / out-of-range cohort**: use a plain four-digit integer between 1900 and
  the current year.

```python
from lactuca import LifeTable

# Each ✔ / ✘ pair below shows independent alternative calls, not a sequence.

# PASEM2020_Rel_1o is a static (period) table — cohort not expected:
lt = LifeTable("PASEM2020_Rel_1o", "m")                  # ✔  no cohort
lt = LifeTable("PASEM2020_Rel_1o", "m", cohort=1975)     # ✘ → ValueError: static table

# PER2020_Ind_1o is a generational table — cohort is required:
lt_gen = LifeTable("PER2020_Ind_1o", "f", cohort=1980)   # ✔  cohort provided
lt_gen = LifeTable("PER2020_Ind_1o", "f")                # ✘ → ValueError: no cohort
```

:::{tip}
Check {doc}`user_guide/bundled_tables` to confirm whether a specific bundled table is
generational (requires `cohort=`) or static (no `cohort` needed).
:::

---

(configuration-errors)=
## Configuration errors

These errors are raised when a configuration setting receives an invalid value.
Configuration is validated at the point of assignment, so the error is
immediate and traceable to the offending line.

(invalid-setting-value)=
### Invalid setting value

**Exception**: `ValueError`

**Message pattern**: `"{setting} must be one of {allowed_values}"`

**Cause**: A direct property assignment (e.g. `config.lx_interpolation = "..."`) or a
`config.set(key, value)` call received a value outside the allowed set for that setting.

**Fix**: Use one of the allowed values from the table below.  String values are
case-sensitive: `"linear"` is valid, but `"Linear"` and `"UDD"` raise; similarly
`"exponential"` is valid but not `"Exponential"` or `"constant_force"`.  Numeric
settings such as `days_per_year` also reject string arguments (e.g. `"365.25"`).

| Setting | Allowed values | Default | Guide |
|---|---|---|---|
| `calculation_mode` | `"discrete_precision"`, `"discrete_simplified"`, `"continuous_precision"`, `"continuous_simplified"` | `"discrete_precision"` | {doc}`user_guide/calculation_modes` |
| `lx_interpolation` | `"linear"`, `"exponential"` | `"linear"` | {doc}`user_guide/lx_interpolation` |
| `force_mortality_method` | `"finite_difference"`, `"spline"`, `"kernel"` | `"finite_difference"` | {doc}`user_guide/force_mortality_methods` |
| `mortality_placement` | `"beginning"`, `"mid"`, `"end"` | `"mid"` | {doc}`user_guide/commutation_functions` |
| `days_per_year` | `360`, `365`, `365.25`, `365.2425`, `366` | `365.25` | {doc}`user_guide/interest_rates_guide` |

**Example — wrong vs. correct assignment:**

```python
from lactuca import config

# Valid — each of these is a correct assignment (run independently, not in sequence):
config.lx_interpolation = "linear"              # ✔  UDD assumption (default)
config.lx_interpolation = "exponential"         # ✔  constant force of mortality
config.calculation_mode = "discrete_precision"  # ✔  valid
config.days_per_year = 365.25                   # ✔  valid
config.reset()                                  # restore all settings to defaults

# Invalid — each raises ValueError independently:
config.lx_interpolation = "udd"                 # ✘ → ValueError: not an allowed value
config.calculation_mode = "exact"               # ✘ → ValueError: not an allowed value
config.days_per_year = 364                      # ✘ → ValueError: 364 not in allowed set
```

(decimal-places)=
### Decimal places — negative or non-integer

**Exception**: `ValueError`

**Message pattern**: `"decimal places must be non-negative integers"`

**Cause**: A `config.decimals.*` setter received a value that is not a non-negative
integer — for example, a negative integer (`-1`), a positive float (`4.5`),
or any other non-integer value.

**Fix**: Use a non-negative integer.  Setting to `0` rounds output to the nearest
integer.  For typical actuarial output use `4`–`6` decimal places; use `10` or higher
only when maximum float64 precision is needed in the output.

```python
from lactuca import config
config.decimals.annuities = 4    # ✔  4 decimal places for annuity output
config.decimals.lx = 8           # ✔  8 decimal places for lx output
config.decimals.annuities = 0    # ✔  round to integer (0 decimal places)
config.decimals.annuities = -1   # ✘ → ValueError: negative integer
config.decimals.annuities = 4.5  # ✘ → ValueError: non-integer float
config.reset()                   # restore all decimals to defaults
```

---

(input-array-errors)=
## Input array errors

These errors occur when two array arguments have incompatible lengths or shapes,
preventing Lactuca from broadcasting or pairing them element-wise.

(mismatched-cashflow-arrays)=
### Mismatched cashflow arrays

**Exception**: `ValueError`

**Message pattern**: `"[{method}] Length of 'cashflow_amounts' must match 'cashflow_times'"`

**Cause**: The two arrays supplied to the irregular cashflow interface have different
lengths (`cashflow_times` and `cashflow_amounts` must have the same length).

**Fix**: Ensure both arrays have the same length before the call.
A common source of mismatch is omitting a payment amount for one cashflow time, or
building the two arrays from different data sources.  Assert equality at the call site:
`assert len(cashflow_times) == len(cashflow_amounts)` before the call.

```python
# cashflow_times and cashflow_amounts are passed to the irregular cashflow interface.
# See user_guide/irregular_cashflows for the full API and calling convention.

# Valid pair — same length, safe to pass to the cashflow method:
cashflow_times   = [1.0, 2.0, 3.0]         # length 3
cashflow_amounts = [100.0, 200.0, 300.0]    # length 3  ✔  lengths match

# Invalid pair — different lengths, raises ValueError when passed:
cashflow_times   = [1.0, 2.0, 3.0]         # length 3
cashflow_amounts = [100.0, 200.0]           # length 2  ✘ → ValueError
```

(broadcasting-errors)=
### Broadcasting errors — incompatible shapes

**Exception**: `ValueError`

**Message pattern**:
```
Incompatible shapes for 'x' and 'x0': {shape1} and {shape2}.
Both must have the same shape or one must have size 1.
```

`{shape1}` and `{shape2}` are NumPy shape tuples, e.g. `(3,)` and `(2,)`.

**Cause**: `InterestRate.vx(x, x0)` computes discount factors at ages `x` relative to
reference age(s) `x0`.  Lactuca broadcasts a length-1 (or scalar) `x0` against `x` of
any length.  If both `x` and `x0` have more than one element but differ in length,
broadcasting fails and this error is raised.

**Fix**: Use equal-length arrays, or pass a scalar or single-element array for `x0`
(either broadcasts freely against an `x` array of any length).

```python
import numpy as np
from lactuca import InterestRate

ir = InterestRate(terms=[5], rates=[0.02, 0.03])  # non-constant rate curve

ages = np.array([30, 40, 50])           # length 3
ir.vx(ages, x0=0)                       # ✔  scalar x0 broadcasts freely
ir.vx(ages, x0=np.array([0]))           # ✔  length-1 array also broadcasts
ir.vx(ages, x0=np.array([0, 0, 0]))    # ✔  same length as ages

x0_bad = np.array([0, 0])               # length 2 ≠ length 3
ir.vx(ages, x0=x0_bad)                  # ✘ → ValueError: incompatible shapes
```

---

(cashflow-utility-errors)=
## Cashflow utility errors

These errors are raised by :func:`generate_payment_times`, :func:`tiered_amounts`, and
:meth:`GrowthRate.amounts` when payment schedule or tier arguments are invalid.

(invalid-payment-frequency)=
### Invalid payment frequency

**Exception**: `ValueError`

**Message patterns**:

| Function | Message pattern |
|---|---|
| `generate_payment_times` | `"[generate_payment_times] Parameter m (payments per year) must be one of {allowed} (value: {value})"`  |
| `GrowthRate.amounts` | `"[GrowthRate.amounts] m must be one of {allowed}, got {value}."` |

**Cause**: The `m` argument is not one of the allowed payment frequencies:
1, 2, 3, 4, 6, 12, 14, 24, 26, 52, or 365.

**Fix**: Use an integer from the allowed set.  Common values: 1 (annual), 2 (semi-annual),
4 (quarterly), 12 (monthly), 52 (weekly).

```python
from lactuca import generate_payment_times as gpt, GrowthRate

times = gpt(n=3, m=12)                         # ✔  monthly
gpt(n=3, m=365)                               # ✔  daily
gpt(n=3, m=7)                                 # ✘ → ValueError: 7 not in allowed set

gr = GrowthRate(0.02)
gr.amounts(times, start=1000.0, m=12)         # ✔
gr.amounts(times, start=1000.0, m=7)          # ✘ → ValueError
```

(invalid-selected-periods)=
### Invalid selected periods

**Exception**: `ValueError` or `TypeError`

**Message patterns**:

| Condition | Exception | Message pattern |
|---|---|---|
| Period value outside `[1, m]` | `ValueError` | `"[generate_payment_times] selected_periods must be integers in [1, {m}] for m={m}."` |
| Non-integer value (e.g. float) | `TypeError` | (raised by the input validator) |

**Cause**: Each element of `selected_periods` must be an integer between 1 and `m`
inclusive.  Periods are 1-based: period 1 is the first payment of the year, period `m`
is the last.  Float values raise `TypeError`; integers outside `[1, m]` raise `ValueError`.

**Fix**: Use integer-valued periods in the range `[1, m]`.

```python
from lactuca import generate_payment_times as gpt

gpt(n=5, m=4, selected_periods=[1, 3])       # ✔  Q1 and Q3
gpt(n=5, m=12, selected_periods=[1, 7])      # ✔  January and July
gpt(n=5, m=4, selected_periods=[0, 3])       # ✘ → ValueError: 0 < 1 (periods are 1-based)
gpt(n=5, m=4, selected_periods=[5])          # ✘ → ValueError: 5 > m=4
gpt(n=5, m=4, selected_periods=[1.5])        # ✘ → TypeError: non-integer value
```

(tier-structure-errors)=
### Tier structure errors

**Exception**: `ValueError`

**Message patterns**:

| Condition | Message pattern |
|---|---|
| Length mismatch | `"[tiered_amounts] len(values) must equal len(breakpoints) + 1, got len(values)={v}, len(breakpoints)={b}."` |
| Breakpoints not strictly ascending | `"[tiered_amounts] breakpoints must be strictly ascending."` |

**Cause**: `tiered_amounts` requires exactly one `values` entry per tier.  With
`len(breakpoints) = B` boundaries there are `B + 1` tiers, so `len(values)` must equal
`B + 1`.  Breakpoints must also be strictly increasing — duplicate or decreasing values
are rejected.

**Fix**: Ensure `len(values) == len(breakpoints) + 1` and sort breakpoints in strictly
ascending order.

```python
import numpy as np
from lactuca import tiered_amounts

times = np.arange(1.0, 11.0)
tiered_amounts(times, breakpoints=[5],    values=[1.0, 1.1])         # ✔  1 bp → 2 values
tiered_amounts(times, breakpoints=[5, 8], values=[1.0, 1.1, 1.2])   # ✔  2 bp → 3 values
tiered_amounts(times, breakpoints=[5],    values=[1.0])              # ✘ → ValueError: 1 ≠ 2
tiered_amounts(times, breakpoints=[5],    values=[1.0, 1.1, 1.2])   # ✘ → ValueError: 3 ≠ 2
tiered_amounts(times, breakpoints=[8, 5], values=[1.0, 1.1, 1.2])   # ✘ → ValueError: not ascending
```

(non-finite-cashflow-inputs)=
### Non-finite cashflow inputs

**Exception**: `ValueError`

**Message patterns**:

| Input | Function | Message pattern |
|---|---|---|
| `times` | `tiered_amounts` | `"[tiered_amounts] times contains non-finite value at index {i}."` |
| `times` | `GrowthRate.amounts` | `"[GrowthRate.amounts] times contains non-finite value at index {i}."` |
| `breakpoints` | `tiered_amounts` | `"[tiered_amounts] breakpoints contains non-finite value at index {i}."` |
| `values` | `tiered_amounts` | `"[tiered_amounts] values contains non-finite value at index {i}."` |
| `start` | `GrowthRate.amounts` | `"[GrowthRate.amounts] start must be finite, got {value}."` |

**Cause**: A NaN or infinite value was found in an input array or in the scalar `start`
argument.  Checks are vectorized and report the index of the first offending element.

**Fix**: Replace or filter non-finite values before calling.  Use `np.isfinite(arr)` to
locate them.

```python
import numpy as np
from lactuca import tiered_amounts, GrowthRate, generate_payment_times as gpt

times = gpt(n=10, m=1)
tiered_amounts(times, [5], [1.0, 1.1])                    # ✔

bad_times = times.copy()
bad_times[3] = np.nan
tiered_amounts(bad_times, [5], [1.0, 1.1])                # ✘ → ValueError: index 3

gr = GrowthRate(0.02)
gr.amounts(times, start=float('inf'), m=1)                # ✘ → ValueError: start not finite
gr.amounts(times, start=float('nan'), m=1)                # ✘ → ValueError: start not finite
```

(times-is-none)=
### times is None

**Exception**: `TypeError`

**Message pattern**: `"[GrowthRate.amounts] times must be array-like, got None."`

**Cause**: `None` was passed as the `times` argument to `GrowthRate.amounts`.  Unlike
other invalid inputs, `None` raises `TypeError` (not `ValueError`) because it is not
array-like at all.

**Fix**: Pass an array-like of payment times — typically the output of
:func:`generate_payment_times`, a NumPy array, or a Python list of floats.

```python
from lactuca import GrowthRate, generate_payment_times as gpt

gr = GrowthRate(0.02)
times = gpt(n=5, m=12)

gr.amounts(times, start=1000.0, m=12)   # ✔
gr.amounts(None,  start=1000.0, m=12)   # ✘ → TypeError: times is None
```

---

(growthrate-errors)=
## GrowthRate errors

These errors are raised by the `GrowthRate` class — its constructor, property setters, and
public methods.

(growthrate-constructor-errors)=
### GrowthRate constructor errors

**Exception**: `ValueError` or `TypeError`

**Message patterns**:

| Condition | Exception | Message pattern |
|---|---|---|
| Invalid `growth_type` | `ValueError` | `"[GrowthRate] growth_type must be 'g' (geometric) or 'a' (arithmetic), got {value!r}."` |
| `apply_from_first` not `bool` | `TypeError` | `"[GrowthRate] apply_from_first must be a bool, got {type}."` |
| `rate` is `bool` | `TypeError` | `"[GrowthRate] rate must be a numeric float, not bool."` |
| `rate` not numeric | `TypeError` | `"[GrowthRate] rate must be a numeric scalar, got {type}."` |
| Geometric `rate ≤ -1` | `ValueError` | `"[GrowthRate] Geometric growth requires rate > -1.0, got {value}."` |
| Arithmetic `rate ≤ -1` | `ValueError` | `"[GrowthRate] Arithmetic growth rate must be > -1.0, got {value}. Factor at period 1 would be non-positive."` |
| Piecewise geom rates ≤ -1 | `ValueError` | `"[GrowthRate] Geometric growth requires all rates > -1.0. Got {value!r} at index {i}."` |
| Arithmetic cumulative factor ≤ 0 | `ValueError` | `"[GrowthRate] Arithmetic growth: cumulative factor at period {k} is non-positive (1 + sum = {v:.6g}). Rates may be too negative."` |
| `rates` without `terms` | `ValueError` | `"[GrowthRate] 'terms' must be provided together with 'rates' for piecewise construction."` |
| `rates` empty | `ValueError` | `"[GrowthRate] 'rates' must not be empty."` |
| `terms` empty | `ValueError` | `"[GrowthRate] 'terms' must not be empty."` |
| `len(rates) ≠ len(terms)+1` | `ValueError` | `"[GrowthRate] len(rates)={n} must equal len(terms)+1={m}."` |
| Term ≤ 0 | `ValueError` | `"[GrowthRate] All terms must be strictly positive integers. Got {value!r} at index {i}."` |
| Scenario dict empty | `ValueError` | `"[GrowthRate] Scenarios dict must not be empty."` |
| Scenario name not string | `ValueError` | `"[GrowthRate] Scenario names must be strings, got {type}."` |
| Scenario value is `bool` | `TypeError` | `"[GrowthRate] Scenario value for {name!r} must be float or GrowthRate, not bool."` |
| Nested multi-scenario | `ValueError` | `"[GrowthRate] Nested multi-scenario GrowthRate not allowed (scenario {name!r})."` |
| Scenario value wrong type | `ValueError` | `"[GrowthRate] Scenario value for {name!r} must be float or GrowthRate, got {type}."` |
| No valid constructor branch | `ValueError` | `"[GrowthRate] Provide either a scalar rate, rates+terms, or a dict of scenarios."` |

**Cause**: The `GrowthRate` constructor validates every input before storing state.
The three valid constructor paths are:

1. **Scalar constant** `GrowthRate(rate=0.02)` — single rate.
2. **Piecewise** `GrowthRate(rates=[0.01, 0.02], terms=[5])` — `len(rates)` must be `len(terms) + 1` and all terms ≥ 1.
3. **Scenario dict** `GrowthRate({'base': 0.02, 'stress': 0.04})` — each value is a float or a simple (non-multi-scenario) `GrowthRate`.

Geometric growth requires every rate `> -1` so the accumulation factor remains positive.
Arithmetic growth also requires every individual rate `> -1` (factor at period 1 > 0)
and, for piecewise schedules, that no prefix sum drives the cumulative factor to 0 or below.

**Fix**: Use one of the three valid paths.  Check `growth_type` is `'g'` or `'a'`;  pass
a `bool` for `apply_from_first`; do not pass `True`/`False` as a rate value.

```python
from lactuca import GrowthRate

# ✔ Correct forms
GrowthRate(0.02)                                           # scalar geometric
GrowthRate(0.02, growth_type='g')                          # explicit geometric
GrowthRate(0.05, growth_type='a')                          # arithmetic
GrowthRate(rates=[0.01, 0.02, 0.03], terms=[5, 10])        # piecewise, 3 rates 2 terms
GrowthRate({'base': 0.02, 'stress': 0.04})                 # scenario dict

# ✘ ValueError / TypeError
GrowthRate(0.02, growth_type='x')                          # ✘ unknown growth_type
GrowthRate(0.02, apply_from_first=1)                       # ✘ int, not bool
GrowthRate(True)                                           # ✘ bool, not float
GrowthRate(-1.0)                                           # ✘ rate ≤ -1
GrowthRate(rates=[0.01, 0.02])                             # ✘ terms not provided
GrowthRate(rates=[0.01, 0.02], terms=[5, 10])              # ✘ 2 rates, needs 3 for 2 terms
GrowthRate(rates=[0.01, 0.02, 0.03], terms=[5, 0])         # ✘ term = 0
GrowthRate({})                                             # ✘ empty dict
```

(growthrate-property-setters)=
### GrowthRate property setters

**Exception**: `ValueError`

**Message patterns**:

| Property | Condition | Message pattern |
|---|---|---|
| `rates` | Multi-scenario container | `"[GrowthRate.rates.setter] Cannot set rates on a multi-scenario container. Set rates on individual scenario instances instead."` |
| `rates` | Not 1-D | `"[GrowthRate.rates.setter] rates must be a 1-D sequence of numerics."` |
| `rates` | Length mismatch | `"[GrowthRate.rates.setter] rates length must match the existing schedule ({n}), got {m}. To change segment count, create a new GrowthRate instance."` |
| `terms` | Multi-scenario container | `"[GrowthRate.terms.setter] Cannot set terms on a multi-scenario container. Set terms on individual scenario instances instead."` |
| `terms` | Constant instance | `"[GrowthRate.terms.setter] Cannot set terms on a constant GrowthRate. Create a new piecewise instance instead."` |
| `terms` | Not 1-D | `"[GrowthRate.terms.setter] terms must be a 1-D sequence of integers."` |
| `terms` | Length mismatch | `"[GrowthRate.terms.setter] terms length must match the existing schedule ({n}), got {m}. To change segment count, create a new GrowthRate instance."` |
| `active_scenario` | Name not found | `"[GrowthRate] Scenario {name!r} not found. Available: {list}."` |

**Cause**: The `rates` and `terms` setters allow in-place updates to an existing schedule
without changing the number of segments.  They are not available on constant instances
(which have no `terms`) or multi-scenario containers.  The `active_scenario` setter
requires the name to already exist in `scenarios`.

**Fix**: Create a new `GrowthRate` instance to change the segment count.  To update rates
in-place, the replacement array must have the same length as the existing `rates` array.

```python
from lactuca import GrowthRate

gr = GrowthRate(rates=[0.01, 0.02], terms=[5])
gr.rates = [0.015, 0.025]          # ✔  same length (2)
gr.terms = [8]                     # ✔  same length (1)

gr.rates = [0.01, 0.02, 0.03]     # ✘ → ValueError: length mismatch
gr.terms = [3, 7]                  # ✘ → ValueError: length mismatch

constant_gr = GrowthRate(0.02)
constant_gr.terms = [5]            # ✘ → ValueError: constant instance

scen_gr = GrowthRate({'base': 0.02})
scen_gr.rates = [0.02]             # ✘ → ValueError: multi-scenario container
scen_gr.active_scenario = 'x'     # ✘ → ValueError: scenario not found
```

(growthrate-add-scenario-errors)=
### GrowthRate.add_scenario errors

**Exception**: `TypeError` or `ValueError`

**Message patterns**:

| Condition | Exception | Message pattern |
|---|---|---|
| `scenario` not a `GrowthRate` | `TypeError` | `"[GrowthRate.add_scenario] scenario must be a GrowthRate instance, got {type}."` |
| Nested multi-scenario | `ValueError` | `"[GrowthRate.add_scenario] Nested multi-scenario GrowthRate instances are not allowed."` |

**Cause**: Scenarios must be simple (constant or piecewise) `GrowthRate` instances.
Passing any other type — or a `GrowthRate` that is itself a multi-scenario container —
raises.

**Fix**: Pass a constant or piecewise `GrowthRate` directly.

```python
from lactuca import GrowthRate

gr = GrowthRate({'base': 0.02})
gr.add_scenario('stress', GrowthRate(0.04))              # ✔
gr.add_scenario('nested', GrowthRate({'a': 0.02}))       # ✘ → ValueError: nested
gr.add_scenario('bad', 0.04)                             # ✘ → TypeError: not GrowthRate
```

(growthrate-factor-errors)=
### GrowthRate.factor errors

**Exception**: `TypeError`

**Message pattern**: `"[GrowthRate.factor] t must be numeric (int, float, or array), got None."`

**Cause**: `None` was passed as `t`.  Unlike invalid numeric values (which raise
`ValueError`), `None` raises `TypeError` because it is not numeric at all.

**Fix**: Pass an integer or array of non-negative integers.

```python
from lactuca import GrowthRate

gr = GrowthRate(0.02)
gr.factor(5)          # ✔
gr.factor(None)       # ✘ → TypeError
```

---

(see-also)=
## See also

**Age and term**
- {doc}`user_guide/commutation_functions` — integer-age requirement; all commutation function signatures
- {doc}`user_guide/lx_interpolation` — `lx_interpolation` allowed values (UDD vs. constant force of mortality)
- {doc}`user_guide/numerical_precision` — float64 tolerance and integer detection behaviour

**Interest rates**
- {doc}`user_guide/interest_rates_guide` — valid `InterestRate` inputs and piecewise construction

**Tables**
- {doc}`user_guide/using_tables` — table construction, sex codes, and loading lifecycle
- {doc}`user_guide/building_tables` — custom `.ltk` format and `qx[omega] = 1.0` requirement
- {doc}`user_guide/bundled_tables` — bundled tables, generational vs. static classification

**Configuration and output**
- {doc}`user_guide/decimals_rounding` — `config.decimals.*` settings and rounding behaviour
- {doc}`user_guide/calculation_modes` — `calculation_mode` allowed values and their behaviour
- {doc}`user_guide/configuration` — all configuration options, allowed values, and defaults
- {doc}`user_guide/force_mortality_methods` — `force_mortality_method` config setting: `"finite_difference"`, `"spline"`, and `"kernel"` methods for estimating $\mu(x)$

**Cashflows and growth rates**
- {doc}`user_guide/irregular_cashflows` — irregular cashflow interface and array alignment
- {doc}`user_guide/growth_rates_guide` — `GrowthRate` construction, `amounts()`, and payment frequency conventions

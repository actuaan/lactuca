(errors-reference)=
# Error Reference

This page is the definitive reference for `ValueError` and `TypeError` exceptions raised
by Lactuca's public API.  For each condition you will find the exact **message pattern**,
the **cause**, a concrete **fix**, and **code examples** showing the triggering call
alongside the correct alternative.

:::{important}
**Scope of this reference** — this page covers *non-obvious*, *cross-cutting* errors
(affecting multiple methods or components), errors that arise from
*configuration-of-state surprises*, and **license enforcement errors** that may be
raised at `import` time or during a long-running session.  Simple parameter validation
whose error message is self-explanatory (e.g. *"n must be non-negative"*) is
intentionally omitted.
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
| **Table modification** | `ValueError` | [Orphan `combination_mode`](#table-modification-combination-mode) | `combination_mode` without `table_combination` in the same dict |
| **Table modification** | `ValueError` | [Invalid `combination_mode`](#table-modification-combination-mode) | Value not exactly `"independent"` or `"udd"` |
| **Table modification** | `ValueError` | [UDD with too many causes](#table-modification-combination-mode) | `combination_mode="udd"` with host + 3+ other tables |
| **Table modification** | `ValueError` | [Host or duplicate in `others`](#host-or-duplicate-instance-in-others) | Same instance as host or twice in `others` |
| **Table modification** | `ValueError` | [Combined decrement exceeds 1.0](#combined-decrement-exceeds-10) | Invalid rates before product formula |
| **Table modification** | `ValueError` | [Decrement rate outside [0, 1] before combine](#decrement-rate-outside-01-before-combine) | Host or other `_decrement` out of range before merge |
| **Configuration** | `ValueError` | [Invalid setting value](#invalid-setting-value) | Disallowed value for a config setting |
| **Configuration** | `ValueError` | [Decimal places](#decimal-places) | Negative or non-integer value for `config.decimals.*` |
| **Input arrays** | `ValueError` | [Cashflow arrays](#mismatched-cashflow-arrays) | `cashflow_times` and `cashflow_amounts` differ in length |
| **Input arrays** | `ValueError` | [Broadcasting](#broadcasting-errors) | `x` and `x0` arrays passed to `InterestRate` have incompatible shapes |
| **Interest rate** | `TypeError` | [Wrong type for `interest_rate`](#wrong-type-interest-rate) | `int` or other non-`float` passed to the `interest_rate` property setter or constructor |
| **Cashflow utilities** | `ValueError` | [Invalid payment frequency](#invalid-payment-frequency) | Invalid `m` passed to `payment_times` or `GrowthRate.amounts` |
| **Cashflow utilities** | `ValueError` | [Invalid selected periods](#invalid-selected-periods) | Period value outside `[1, m]` in `payment_times` |
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
| **License** | `LactucaLicenseError` | [License errors — overview](#license-errors) | Any licensing failure (catch-all base class) |
| **License** | `LicenseExpiredError` | [License expired](#license-expired) | Stored or online validation confirms the license period has ended |
| **License** | `LicenseRevokedError` | [License revoked or suspended](#license-revoked) | License server reports revoked or suspended status |
| **License** | `LicenseInvalidError` | [Invalid license or device binding](#license-invalid) | Unrecognized key, unexpected server response, network failure during activation, or malformed license data |
| **License** | `LicenseInvalidError` | [Device pool full (LAC-1013)](#lac-1013-device-pool-full) | Maximum activated devices for the plan already registered on the license server |
| **License** | `LicenseInvalidError` | [Cross-device fingerprint mismatch (LAC-2002)](#lac-2002-cross-device) | Local license file was created on another machine, or JSON copied between hosts |
| **License** | `ActivationRequiredError` | [Activation required](#activation-required) | No local license file in non-interactive environment, missing `LACTUCA_LICENSE_KEY` in CI, or cancelled/incomplete activation |
| **License** | `LicenseSeatExhaustedError` | [Concurrent session limit (LAC-4001)](#license-seat-exhausted) | Maximum simultaneous Python processes for the plan already in use |
| **License** | `LicenseSeatExhaustedError` | [Offline seat grace expired (LAC-4003)](#lac-4003-offline-grace) | Seat availability could not be verified within the offline grace period |
| **License** | `LicenseTamperedError` | [Missing signature fields (LAC-3001)](#lac-3001-missing-signature) | Local license file missing required signature fields — auto-recoverable using stored key |
| **License** | `LicenseTamperedError` | [Invalid signature (LAC-3002)](#lac-3002-invalid-signature) | Cryptographic signature verification failed — auto-recoverable using stored key |
| **License** | `LicenseTamperedError` | [Integrity field missing (LAC-3003)](#lac-3003-mac-missing) | Local license file missing device-bound integrity field — auto-recoverable using stored key |
| **License** | `LicenseTamperedError` | [Integrity check failed (LAC-3004)](#lac-3004-mac-mismatch) | Protected fields modified after writing, or file copied from another device — auto-recoverable using stored key |
| **License** | `LicenseTamperedError` | [Clock rollback (LAC-3005)](#lac-3005-clock-rollback) | System clock moved back past the last validated date |
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
The license rows raise subclasses of `LactucaLicenseError` (not `ValueError` or
`TypeError`).  See [License errors](#license-errors) for import-time behaviour and CI handling.
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
from lactuca import InterestRate

ir = InterestRate(0.03)
ir.vx(5)                                  # ✔  x0=0 by default
ir.vx([30, 40], x0=25)                    # ✔
ir.vx(-1)                                 # ✘ → ValueError: negative x
ir.vx(5, x0=-1)                           # ✘ → ValueError: negative x0
ir.vx([10, 20], x0=[15, 5])               # ✘ → ValueError: x[0] < x0[0]
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
only deviations $> 10^{-4}$ trigger this error.  If you need a rate below $1.0$ at the
last age of interest, extend $\omega$ by one year and store your intended rate at
$\omega - 1$, with $q_\omega = 1.0$ on the new terminal row.

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
config.reset_to_defaults()                      # restore all settings to defaults

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
config.reset_to_defaults()       # restore all decimals to defaults
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
from lactuca import InterestRate

ir = InterestRate(terms=[5], rates=[0.02, 0.03])  # non-constant rate curve

ages = [30, 40, 50]                     # length 3
ir.vx(ages, x0=0)                       # ✔  scalar x0 broadcasts freely
ir.vx(ages, x0=[0])                     # ✔  length-1 list also broadcasts
ir.vx(ages, x0=[0, 0, 0])              # ✔  same length as ages

x0_bad = [0, 0]                         # length 2 ≠ length 3
ir.vx(ages, x0=x0_bad)                  # ✘ → ValueError: incompatible shapes
```

---

(cashflow-utility-errors)=
## Cashflow utility errors

These errors are raised by :func:`payment_times`, :func:`tiered_amounts`, and
:meth:`GrowthRate.amounts` when payment schedule or tier arguments are invalid.

(invalid-payment-frequency)=
### Invalid payment frequency

**Exception**: `ValueError`

**Message patterns**:

| Function | Message pattern |
|---|---|
| `payment_times` | `"[payment_times] Parameter m (payments per year) must be one of {allowed} (value: {value})"`  |
| `GrowthRate.amounts` | `"[GrowthRate.amounts] m must be one of {allowed}, got {value}."` |

**Cause**: The `m` argument is not one of the allowed payment frequencies:
1, 2, 3, 4, 6, 12, 14, 24, 26, 52, or 365.

**Fix**: Use an integer from the allowed set.  Common values: 1 (annual), 2 (semi-annual),
4 (quarterly), 12 (monthly), 52 (weekly).

```python
from lactuca import payment_times, GrowthRate

times = payment_times(n=3, m=12)                         # ✔  monthly
payment_times(n=3, m=365)                               # ✔  daily
payment_times(n=3, m=7)                                 # ✘ → ValueError: 7 not in allowed set

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
| Period value outside `[1, m]` | `ValueError` | `"[payment_times] selected_periods must be integers in [1, {m}] for m={m}."` |
| Non-integer value (e.g. float) | `TypeError` | (raised by the input validator) |

**Cause**: Each element of `selected_periods` must be an integer between 1 and `m`
inclusive.  Periods are 1-based: period 1 is the first payment of the year, period `m`
is the last.  Float values raise `TypeError`; integers outside `[1, m]` raise `ValueError`.

**Fix**: Use integer-valued periods in the range `[1, m]`.

```python
from lactuca import payment_times

payment_times(n=5, m=4, selected_periods=[1, 3])       # ✔  Q1 and Q3
payment_times(n=5, m=12, selected_periods=[1, 7])      # ✔  January and July
payment_times(n=5, m=4, selected_periods=[0, 3])       # ✘ → ValueError: 0 < 1 (periods are 1-based)
payment_times(n=5, m=4, selected_periods=[5])          # ✘ → ValueError: 5 > m=4
payment_times(n=5, m=4, selected_periods=[1.5])        # ✘ → TypeError: non-integer value
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
from lactuca import tiered_amounts, payment_times

times = payment_times(n=10, m=1)
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
from lactuca import tiered_amounts, GrowthRate, payment_times

times = payment_times(n=10, m=1)
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
:func:`payment_times`, a NumPy array, or a Python list of floats.

```python
from lactuca import GrowthRate, payment_times

gr = GrowthRate(0.02)
times = payment_times(n=5, m=12)

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

**Licensing**
- {doc}`activation` — activation flow, device-bound `license.json`, and CLI commands
- {doc}`faq_licensing` — device transfer, copying `license.json`, concurrent session limits, and offline grace periods

---

(license-errors)=
## License errors

License errors are raised during `import lactuca` or during a long-running session
(session keep-alive).  They are subclasses of `LactucaLicenseError` and are **not**
`ValueError` or `TypeError`.

Import the exception types from the top-level package or from `lactuca.exceptions`:

```python
from lactuca import LicenseExpiredError
# equivalent:
from lactuca.exceptions import LicenseExpiredError
```

:::{important}
**Import-time behaviour** — on `import lactuca`, the package initializer converts
`LactucaLicenseError` subclasses to a clean `SystemExit` (message on stderr, no Python
traceback).  A surrounding ``except LicenseExpiredError`` (or any other subclass)
**will not** catch import-time failures.

Use one of these approaches instead:

- **CI / automation:** set ``LACTUCA_LICENSE_KEY`` before import (see {doc}`activation`).
- **Subprocess:** run ``python -c "import lactuca"`` and inspect the exit code / stderr.
- **Interactive recovery:** ``python -m lactuca activate`` or ``python -m lactuca license doctor``.
- **Automation guard:** catch ``SystemExit`` and inspect the message string for LAC codes
  (see examples below).

The exception types below are the **typed errors** raised by the activation layer; their
messages appear in CLI output and in the ``SystemExit`` string on import.

Typed errors may include a ``Server-status:`` diagnostic line (for example
``EXPIRED``, ``SUSPENDED``, ``NOT_FOUND``).  Include the full message when
contacting support; you do not need to interpret the status code yourself.
:::

### Exception overview

| Exception | When it is raised |
|---|---|
| `LactucaLicenseError` | Catch-all base — handle any licensing error in one ``except`` block |
| `LicenseExpiredError` | Stored expiry date or online validation confirms the license period has ended |
| `LicenseRevokedError` | License server reports revoked, suspended, or permanently deleted status |
| `LicenseInvalidError` | Unrecognized key, fingerprint mismatch, device activation limit, unexpected server response, network failure during activation, or malformed license data |
| `LicenseTamperedError` | Local license file failed integrity or signature checks, or system clock rollback detected |
| `ActivationRequiredError` | No valid license and activation is required (non-interactive import, missing env key in CI, cancelled activation) |
| `LicenseSeatExhaustedError` | Concurrent session limit reached **or** offline seat grace period expired |

(license-expired)=
### License expired — `LicenseExpiredError`

**Exception**: `LicenseExpiredError`

**Message pattern** (representative):
```
[LAC-1011] Your license has expired.
Action: Purchase a new license at: {pricing_url}
```

Local expiry (without a fresh online check) uses code **LAC-2001**; online revalidation
uses **LAC-2003**.

**Cause**: The stored expiry date or an online validation confirms the license period
has ended.

**Fix**: Renew at [lactuca.io/pricing](https://lactuca.io/pricing), then re-run
`import lactuca` or `python -m lactuca license refresh`.  In CI, update
``LACTUCA_LICENSE_KEY`` to the renewed key if the stored file cannot be updated
automatically.

(license-revoked)=
### License revoked or suspended — `LicenseRevokedError`

**Exception**: `LicenseRevokedError`

**Message pattern** (representative):
```
[LAC-1012] Your license has been revoked by the license server.
Action: Contact support to resolve this issue.
```

Online revalidation uses **LAC-2004**.

**Cause**: The license server reports that the license has been revoked, suspended,
or permanently deleted (typically after payment failure, subscription cancellation,
or manual action by Lactuca support).

**Fix**: Contact [support@lactuca.io](mailto:support@lactuca.io) with the full error
message.  Do not expect automatic recovery after a permanent deletion on the server;
a new license key is required.

(license-invalid)=
### Invalid license or device binding — `LicenseInvalidError`

**Exception**: `LicenseInvalidError`

Catch-all for license-key and device-binding failures that are not expiry, revocation,
tampering, or concurrent-session errors.  The LAC code in the message identifies the
specific scenario.

| LAC code | Topic | Detail |
|---|---|---|
| **LAC-1013** | Device pool full | [Device pool full (LAC-1013)](#lac-1013-device-pool-full) |
| **LAC-2002** | Cross-device use | [Cross-device fingerprint mismatch (LAC-2002)](#lac-2002-cross-device) |
| **LAC-1015** | Fingerprint scope | [Fingerprint scope mismatch (LAC-1015)](#lac-1015-fingerprint-scope) |
| **LAC-1014** | Device registration failed | [Device registration failed (LAC-1014)](#lac-1014-device-registration) |
| **LAC-1016** | Activation incomplete | Server did not receive the initial device ping |
| **LAC-1017** | Network error | Interactive activation blocked by connectivity |
| **LAC-1018** | Unexpected server status | Key rejected or status not recognized during activation |
| **LAC-2005** | Online revalidation | Malformed payload or blocking `last_server_status` while offline |
| **LAC-4002** | Process lease | Unexpected failure acquiring a concurrent session lease |
| **LAC-5001** | Server configuration | Client/server configuration mismatch |

**Fix**: Read the LAC code and ``Action:`` line in the message, then follow the linked
subsection above or run ``python -m lactuca license doctor``.

(lac-1013-device-pool-full)=
### Device pool full — `LicenseInvalidError` (LAC-1013)

**Exception**: `LicenseInvalidError`

**Message pattern**:
```
[LAC-1013] Your license has reached the maximum number of activated devices.
Server-status: TOO_MANY_MACHINES | fingerprint: {fingerprint}
Action: To transfer activation to this device, visit: {faq_device_transfer_url}
If this device was activated with a different license key, deactivate that license first, then free a device slot if needed.
```

**Cause**: The license server refused to register this device because the plan's
**device pool** is full.  Each physical or virtual machine counts as one activated
device (distinct from concurrent Python sessions — see [LAC-4001](#license-seat-exhausted)).

The client raises **LAC-1013** when Keygen returns ``TOO_MANY_MACHINES``, and also
during **first activation** when validate-key reports ``FINGERPRINT_SCOPE_MISMATCH``
but every device slot on **this** license is already in use (pool exhausted after a
failed ``POST /machines``).

Device limits per plan:

| Plan | Activated devices |
|---|---|
| Trial | 1 |
| Individual | 1 |
| Academic & Community | 1 |
| Team | 10 |
| Enterprise | 50 |
| OEM | unlimited |

Typical triggers:

- Running ``python -m lactuca activate`` on a new machine while every slot is already
  in use on the license server.
- First activation when the server returns ``FINGERPRINT_SCOPE_MISMATCH`` but the
  license already has the maximum number of registered machines (Enterprise 50/50,
  Team 10/10, etc.).
- Online revalidation or machine registration after a legitimate device change, before
  an old slot was released.
- After ``license refresh`` records ``TOO_MANY_MACHINES`` in ``last_server_status``,
  the next ``import lactuca`` requires a successful online re-check — import does not
  continue offline while the pool remains full.

**Fix**:

1. **Free a device slot** before activating on this machine — see the
   {ref}`device transfer procedure <device-transfer>` in the Licensing FAQ.
2. If this device was activated with **another license key**, deactivate that license
   on this machine first (then free a slot on the intended license if still at capacity).
3. On Team or Enterprise plans, release an unused device slot (or ask support to remove
   a stale machine registration) before retrying activation on the new host.
4. After a slot is free, run ``python -m lactuca activate`` (or ``python -m lactuca license refresh``)
   while connected to the internet, then ``import lactuca``.

Contact [support@lactuca.io](mailto:support@lactuca.io) if you need a slot released
immediately (subject line: *Device transfer request*).

(lac-2002-cross-device)=
### Cross-device fingerprint mismatch — `LicenseInvalidError` (LAC-2002)

**Exception**: `LicenseInvalidError`

**Message pattern**:
```
[LAC-2002] This device is not activated for your license (hardware fingerprint mismatch).
stored-fingerprint: {stored} | current-fingerprint: {current}
Action: To transfer activation to this device, visit: {faq_device_transfer_url}
```

**Cause**: The local ``license.json`` was written on a **different** machine.  The
stored hardware fingerprint does not match this device.  Common scenarios:

- **Copying ``license.json``** from another PC — not a valid way to share a license
  (see {doc}`faq_licensing`).
- Import on a second host where the file still lists the first machine's fingerprint.
- Online recovery is attempted with a stored key, but the file's fingerprint belongs to
  another device — recovery runs only on the **same** device (LAC-3001/3003/3004);
  cross-host copies raise **LAC-2002** instead.

Lactuca does **not** silently register a new device when the JSON was copied from
elsewhere.

**Fix**:

1. On **this** machine, delete the copied ``license.json`` — do not copy the file from
   the other PC again.
2. If your plan is at its device limit, free a slot first — {ref}`device transfer <device-transfer>`.
3. Run ``python -m lactuca activate`` and enter your license key on **this** device.

See also {doc}`activation` (device-bound license file).

(lac-1014-device-registration)=
### Device registration failed — `LicenseInvalidError` (LAC-1014)

**Exception**: `LicenseInvalidError`

**Message pattern**:
```
[LAC-1014] Device registration failed: the license server could not register this device.
Server-status: {keygen_status} | fingerprint: {fingerprint}
Action: Check your internet connection and try again.
```

**Cause**: The license server did not complete device registration.  Typical triggers:

- ``NO_MACHINE`` / ``NO_MACHINES`` after activation could not create a machine record.
- ``FINGERPRINT_SCOPE_MISMATCH`` during first activation when the client could not
  confirm whether the device pool is full (transient list/count failure on the
  license server).

**Fix**: Verify network connectivity and retry ``python -m lactuca activate``.  If the
error persists, run ``python -m lactuca license doctor`` and contact
[support@lactuca.io](mailto:support@lactuca.io).

(lac-1015-fingerprint-scope)=
### Fingerprint scope mismatch — `LicenseInvalidError` (LAC-1015)

**Exception**: `LicenseInvalidError`

**Message pattern** (interactive activation):
```
[LAC-1015] This device's fingerprint is already registered under a different license key.
Server-status: FINGERPRINT_SCOPE_MISMATCH | fingerprint: {fingerprint}
Action: Deactivate the previous license on this device first, or contact support.
```

**Cause**: During **interactive activation** (``python -m lactuca activate``), the
license server reported that this hardware fingerprint is already bound to a
**different** license key, and the device pool on the key being activated is
**not** full.  This is distinct from:

- **LAC-2002** — local ``license.json`` from another machine (stored fingerprint mismatch).
- **LAC-1013** — device pool full on **this** license (``TOO_MANY_MACHINES``, or
  ``FINGERPRINT_SCOPE_MISMATCH`` with pool at capacity).

**Fix**: Release or deactivate the other license on this device, then activate with the
intended key.  If you believe the binding is stale, contact
[support@lactuca.io](mailto:support@lactuca.io).

:::{note}
**First activation** with ``FINGERPRINT_SCOPE_MISMATCH``: if every device slot on the
license being activated is already in use, the client raises **LAC-1013** (pool full),
not LAC-1015.  If the machine count cannot be retrieved, the client raises **LAC-1014**.

**Online revalidation** (``import lactuca``, ``license refresh``) can receive the same
server status ``FINGERPRINT_SCOPE_MISMATCH`` in other contexts:

- If the stored fingerprint matches this device but the pool is full, the client raises
  **LAC-1013**, not LAC-1015.
- If ``license refresh`` records ``FINGERPRINT_SCOPE_MISMATCH`` or ``TOO_MANY_MACHINES``
  in ``last_server_status``, the next import forces an immediate online re-check.
  While offline, import is blocked with **LAC-2005** citing the stored server status;
  after reconnecting, the client resolves the situation to LAC-1013, LAC-2002, or
  LAC-2005 as appropriate — import does **not** silently succeed with a copied license
  file.
:::

(activation-required)=
### Activation required — `ActivationRequiredError`

**Exception**: `ActivationRequiredError`

**Cause**: No valid license is found and activation is required — for example on first
import when no local license file is present and the environment is non-interactive,
when ``LACTUCA_LICENSE_KEY`` is not set in CI/CD, or when interactive activation was
cancelled or could not complete (trial request, key entry, etc.).

**Fix**:

- **Interactive:** run ``python -m lactuca activate``.
- **CI / headless:** set ``LACTUCA_LICENSE_KEY`` before import (see {doc}`activation`).
- **Trial:** follow the URL in the message or visit [lactuca.io/pricing](https://lactuca.io/pricing).

:::{note}
``ActivationRequiredError`` carries an ``already_printed`` flag.  When the activation
flow already printed guidance to the terminal, import exits with code **0** and no
duplicate message.
:::

(license-seat-exhausted)=
### Concurrent session limit — `LicenseSeatExhaustedError` (LAC-4001)

**Exception**: `LicenseSeatExhaustedError`

**Message pattern**:
```
[LAC-4001] All concurrent sessions for this license are in use.
Action: Close another active Lactuca session, or upgrade your plan: {pricing_url}
```

**Cause**: The maximum number of simultaneous active Python processes allowed by the
license plan is already in use.  The license server refused to grant a new process lease.

Concurrent session limits per tier:

| Plan | Concurrent sessions |
|---|---|
| Trial | 1 |
| Individual | 1 |
| Academic & Community | 1 |
| Team | 10 |
| Enterprise | 50 |
| OEM | unlimited |

**Fix**:

1. **Wait for a seat to free up.** Seats are released automatically when a running
   process exits cleanly. If a process was killed or crashed, the seat is released after
   the heartbeat lease expires (30 minutes for single-user tiers; 60 minutes for
   Team/Enterprise).

2. **Check for stuck processes.** Look for background scripts, Jupyter kernels, or
   scheduled jobs running Lactuca that you may have forgotten about.

3. **Upgrade your plan.** Team (10 sessions) and Enterprise (50 sessions) are suitable
   for server deployments and teams running parallel jobs.

4. **Handle in pipeline scripts** that may run concurrently (inspect ``SystemExit`` on
   import — see {ref}`license-errors`):

```python
try:
    import lactuca
except SystemExit as exc:
    msg = str(exc) or ""
    if "LAC-4001" in msg:
        raise SystemExit("Lactuca seat limit reached. Retry when a session is free.")
    raise
```

:::{note}
The seat is always released on clean Python exit.  If you are using
``multiprocessing``, each child process consumes one seat.  For high-parallelism scenarios
(e.g. 20+ workers) use an Enterprise license or restructure the pipeline to share a
single Lactuca process across workers.
:::

(lac-4003-offline-grace)=
### Offline seat grace expired — `LicenseSeatExhaustedError` (LAC-4003)

**Exception**: `LicenseSeatExhaustedError` (same class as LAC-4001)

**Message pattern**:
```
[LAC-4003] Offline grace period has expired: cannot verify seat availability.
Action: Connect to the internet and try again.
```

**Cause**: Lactuca could not verify seat availability with the license server within the
allowed offline grace period (see {doc}`faq_licensing` for grace duration).  This is
distinct from LAC-4001: the session limit may not be reached, but the process cannot
confirm an available seat while offline.

**Fix**: Connect to the internet and retry ``python -m lactuca`` or ``import lactuca``.  No manual file deletion
is required — the local license file is kept so recovery is retried on the next import.

(lac-3001-missing-signature)=
### Missing signature fields — `LicenseTamperedError` (LAC-3001)

**Exception**: `LicenseTamperedError`

**Message pattern**:
```
[LAC-3001] license.json is missing required signature fields (signed_data or signature).
Cause: The file was modified manually, corrupted, or is from an incompatible version.
Action: Re-run python -m lactuca or import lactuca. Recovery is automatic if you are online.
```

**Cause**: The local license file is missing required signature fields.

**Recovery**: **Auto-recoverable** when a license key is stored in the file.  Lactuca
validates online with the stored key.  If the key is still valid, the local license file
is silently overwritten and import continues.  If the key has expired, a
`LicenseExpiredError` is raised; if revoked, a `LicenseRevokedError`; if the network is
unavailable, an `ActivationRequiredError` explaining how to reconnect.

**Fix**: Re-run ``python -m lactuca`` or ``import lactuca`` while connected to the internet.  In non-interactive
environments, set ``LACTUCA_LICENSE_KEY`` so re-activation can proceed without a prompt.

(lac-3002-invalid-signature)=
### Invalid cryptographic signature — `LicenseTamperedError` (LAC-3002)

**Exception**: `LicenseTamperedError`

**Message pattern**:
```
[LAC-3002] license.json Ed25519 signature verification failed.
Cause: The file was modified after being written, or the signing key has changed.
Action: Re-run python -m lactuca or import lactuca. Recovery is automatic if you are online.
```

**Cause**: The cryptographic signature stored in the local license file does not verify.
This happens when protected payload fields were edited manually, the file was tampered
with, corrupted during an incomplete write, or the signing key has changed.

**Recovery**: **Auto-recoverable** when a license key is stored in the file.  Lactuca
validates online with the stored key.  If the key is still valid, the local license file
is silently overwritten and import continues normally.  If the key has expired, a
`LicenseExpiredError` with the renewal URL is raised instead; if revoked, a
`LicenseRevokedError`; if the network is unavailable, an `ActivationRequiredError`
explaining how to reconnect.

**Fix**: In most cases, simply re-run ``python -m lactuca`` or ``import lactuca`` while connected to the internet.
In non-interactive environments, ensure ``LACTUCA_LICENSE_KEY`` is set so re-activation
can proceed without a prompt if the file needs to be recreated.

```python
import lactuca   # auto-recovers when online and a stored key is present
```

(lac-3003-mac-missing)=
### Integrity field missing — `LicenseTamperedError` (LAC-3003)

**Exception**: `LicenseTamperedError`

**Message pattern**:
```
[LAC-3003] license.json integrity check failed: mac field is missing.
Cause: The file was written by an older Lactuca version, modified manually, or copied from another device.
Action: Re-run python -m lactuca or import lactuca. If recovery does not complete, run
  'python -m lactuca license doctor' and
  'python -m lactuca license refresh'. Delete license.json manually
  only if recovery still fails.
```

**Cause**: The local license file does not contain the device-bound integrity field.
This happens when the file was written by an older Lactuca version, edited manually, or
copied from another device.

**Recovery**: **Auto-recoverable** when a license key is stored in the file.  Lactuca
validates online with the stored key.  If the key is still valid, the local license file
is silently overwritten and import continues normally.  If the key has expired, a
`LicenseExpiredError` with the renewal URL is raised instead; the file is kept.  If the
network is unavailable, an `ActivationRequiredError` explaining how to reconnect is
raised; the file is kept so recovery is retried automatically on the next import.

**Fix**: Simply re-run ``python -m lactuca`` or ``import lactuca`` while connected to the internet.  In
non-interactive environments, ensure ``LACTUCA_LICENSE_KEY`` is set in case the stored key
has expired and a new one must be entered.

```python
import lactuca   # auto-recovers when online and a stored key is present
```

(lac-3004-mac-mismatch)=
### Integrity check failed — `LicenseTamperedError` (LAC-3004)

**Exception**: `LicenseTamperedError`

**Message pattern**:
```
[LAC-3004] license.json integrity check failed: mac mismatch.
Cause: The file was modified after being written, or was copied from a different device.
Action: Re-run python -m lactuca or import lactuca. If recovery does not complete, run
  'python -m lactuca license doctor' and
  'python -m lactuca license refresh'. Delete license.json manually
  only if recovery still fails.
```

**Cause**: The device-bound integrity check failed — protected fields in the local
license file do not match the expected value for this device.  This happens when
protected fields were edited manually (e.g. extending ``expires_at`` by hand) or the file
was copied from another machine (the integrity binding is device-specific).

**Recovery**: **Auto-recoverable** when a license key is stored in the file.  Identical
recovery flow to LAC-3003: Lactuca validates online with the stored key and, if still
valid, silently overwrites the local license file.  The file is never deleted
automatically.

:::{note}
The most common cause of LAC-3004 is manually editing ``expires_at`` to extend a license.
In that case online revalidation returns the server's authoritative expiry date and
raises `LicenseExpiredError` with the renewal URL — no trial is offered.  To extend
your license, renew at [lactuca.io/pricing](https://lactuca.io/pricing) and then
re-run ``python -m lactuca`` or ``import lactuca``; renewal is detected automatically via the stored key.
:::

**Fix**: In non-interactive environments set ``LACTUCA_LICENSE_KEY`` before importing.
Never copy the local license file between devices — each device must activate
independently.

**Operational shortcut**: use CLI diagnostics before manual cleanup:

```bash
python -m lactuca license doctor
python -m lactuca license refresh
```

:::{important}
The local license file is device-bound and cannot be shared between machines.  Each device
consumes one activation slot from the license pool.  To transfer a license to a new
machine, see {ref}`device-transfer` in the Licensing FAQ.
:::

```python
# If expires_at was edited by hand, the integrity check fails (LAC-3004).
# After renewing the same key at lactuca.io/pricing, just re-import:
import lactuca   # detects renewal automatically when online
```

(lac-3005-clock-rollback)=
### Clock rollback — `LicenseTamperedError` (LAC-3005)

**Exception**: `LicenseTamperedError`

**Message pattern**:
```
[LAC-3005] System clock rollback detected.
Cause: The system clock has been moved back past the last validated date.
Action: Restore the system clock to the correct date and time, then retry.
```

**Cause**: The system clock is set to a time more than 60 seconds **before** the last
validated timestamp recorded in the local license file.  Because that timestamp is covered
by the device-bound integrity check (see LAC-3004), it cannot be forged.  Rolling the
clock back is detected as a tampering attempt aimed at re-using an expired license.

**Recovery**: This error is **not auto-recoverable** — Lactuca cannot fix the system
clock on your behalf.

**Fix**: Restore your system clock to the correct date and time, then retry ``python -m lactuca`` or ``import lactuca``.
On Windows you can sync the clock with:

```powershell
w32tm /resync
```

On macOS/Linux, enable and start the NTP daemon:

```bash
# macOS
sudo sntp -sS time.apple.com

# Linux (systemd)
sudo timedatectl set-ntp true
```

:::{note}
The 60-second tolerance exists to absorb normal NTP corrections (typically < 1 second)
without false positives.  Legitimate clock resets of this magnitude are rare; if your
clock legitimately jumped back more than 60 seconds, re-syncing via NTP will resolve it.
:::

(table-modification-combination-mode)=
## Table modification — `combination_mode`

Errors raised by `modify_qx`, `modify_ix`, and `modify_ox` when the
`combination_mode` key is invalid or inconsistent with `table_combination`.

### Orphan `combination_mode`

**Exception**: `ValueError`

**Message pattern**:

```
combination_mode requires table_combination in the same modification dict
```

**Cause**: `combination_mode` was passed without `table_combination` in the same dict.

**Fix**: Include both keys, or omit `combination_mode` (defaults to `independent`).

```python
# Wrong
lt.modify_qx({"combination_mode": "udd"})

# Correct
lt.modify_qx({"table_combination": et, "combination_mode": "udd"})
```

### Invalid `combination_mode` literal

**Exception**: `ValueError`

**Message patterns**:

```
combination_mode must be the string 'independent' or 'udd'
Unknown combination_mode '{value}'; allowed values are 'independent' and 'udd'
```

**Cause**: Value is not exactly `"independent"` or `"udd"` (including legacy
`"udd_2"`, `"udd_3"`, wrong case, booleans, or non-strings).

**Fix**: Use `"independent"` or `"udd"` only.

### UDD with too many causes

**Exception**: `ValueError`

**Message pattern**:

```
combination_mode='udd' is limited to 2 or 3 causes (host plus 1 or 2 other tables); got {n} causes with {m} other table(s).
```

**Cause**: `combination_mode="udd"` with host plus three or more other tables (four+
causes). v1 supports UDD for two or three causes only.

**Fix**: Use `"independent"` for four or more causes, or reduce the number of tables
in `table_combination`.

(host-or-duplicate-instance-in-others)=
### Host or duplicate instance in `others`

**Exception**: `ValueError`

**Message patterns**:

```
table_combination: cannot combine a table with itself.
table_combination: duplicate table instance in the others list.
```

**Cause**: The same `DecrementTable` instance was passed as both host and other
(e.g. `et.modify_ox({"table_combination": et})`), or listed twice in
`others` (e.g. `[et, et]`). Competing-risk formulas would treat the same rates
as multiple independent causes.

**Fix**: Pass distinct instances for each cause. For file/base rates on each,
use separate objects or `reset_modifications()` before combining.

(combined-decrement-exceeds-10)=
### Combined decrement exceeds 1.0

**Exception**: `ValueError`

**Message pattern**:

```
table_combination: combined decrement probability exceeds 1.0 at age {x} (q_combined=...)
table_combination: combined decrement probability exceeds 1.0 at index {i} (calendar age {x} after age_shift={n}) (q_combined=...)
```

**Cause**: Prior keys in the same dict (often `decrement_multiplier`) pushed host
or aligned rates above 1 before the product formula ran.

**Fix**: Use valid rates in $[0, 1]$ at each age, or reorder keys so scaling
does not create invalid intermediates before combine.

(decrement-rate-outside-01-before-combine)=
### Decrement rate outside [0, 1] before combine

**Exception**: `ValueError`

**Message pattern**:

```
table_combination: host decrement rate outside [0, 1] at index {i} (age {x}) (q=...)
table_combination: host decrement rate outside [0, 1] at index {i} (calendar age {x} after age_shift={n}) (q=...)
table_combination: ExitTable decrement rate outside [0, 1] at index {i} (age {x}) (q=...)
```

**Cause**: The host or an *other* table’s active `_decrement` vector contained a
rate below 0 or above 1 before the competing-risk formula ran (often from direct
`_decrement` manipulation or corrupt data). Negative combined rates would
otherwise be clipped silently to 0 at the final safety step.

**Fix**: Restore valid rates in $[0, 1]$ on every table (`reset_modifications()`
or a fresh instance), then combine again.

### Other table shortened by `age_shift`

**Exception**: `ValueError`

**Message pattern**:

```
table_combination: ExitTable was shortened by a prior age_shift modification. Apply age_shift on the table you are modifying (e.g. LifeTable.modify_qx), not on the other table.
```

**Cause**: The table passed to `table_combination` was previously modified with
`age_shift`, so its `_decrement` array is shorter than `_decrement_base`.

**Fix**: Apply `age_shift` on the table you are modifying together with
`table_combination`:

```python
# Wrong — shortens ExitTable, then LifeTable cannot combine
et.modify_ox({"age_shift": 40})
lt.modify_qx({"table_combination": et})

# Correct — shift LifeTable, align exit rates by calendar age
lt.modify_qx({"age_shift": 40, "table_combination": et})
```

See also {doc}`user_guide/modifying_decrements`.


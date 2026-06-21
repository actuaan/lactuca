# Calculation Modes

Lactuca supports four calculation modes that control how life contingency present values are
computed. The active mode is set globally via `config.calculation_mode` and applies equally
to **annuities** (`äx`, `ax`), **insurances** (`Ax`), and **pure endowments** (`nEx`).

(actuarial-coherence-of-modes)=
## Actuarial coherence across modes

Lactuca exposes **four calculation modes**, not four different products. Every mode values the
**same actuarial present value** — annuity-due $\ddot{a}$, insurance $A$, pure endowment
${}_n E$ — under the **same global conventions**:

- Decrement tables and modifications (`modify_qx`, `table_combination`,
  optional `combination_mode` — see {doc}`modifying_decrements`) 
- `config.mortality_placement`, `config.lx_interpolation`, `config.force_mortality_method`
- Interest (`InterestRate`), growth (`GrowthRate`), and payment timing (`äx` vs `ax`)
- Joint-life independence; scalar and batch APIs on the same underlying definitions

**Actuarial coherence at 100% does not mean numerical identity.** Modes implement different
approximation paths (exact payment grid, Woolhouse, numerical integration, adjacent-age
averaging). Differences are expected and actuarially legitimate when the mode is used within
its documented scope.

| What must align across modes | What may differ |
|------------------------------|-----------------|
| Product definition (benefit timing, deferment, term) | Approximation formula per mode |
| Global `config` conventions | Rounded vs unrounded $l_x$ (discrete vs continuous) |
| Same table / `ir` / `gr` inputs | Relative error of simplified vs precision |
| Delegation when an approximation is invalid (e.g. fractional $n$) | Speed and internal grid resolution |

Mode selection is an **accuracy / speed trade-off**, not a switch of actuarial model.
For regulatory and production work, `discrete_precision` is the reference discrete
implementation. Use `discrete_simplified` or continuous modes for sensitivity runs,
benchmarking, or exploratory analysis — compare results against `discrete_precision` rather
than assuming bit-identical output.

:::{important}
Do **not** expect `discrete_precision` and `continuous_precision` to match to the last
decimal place, or `discrete_simplified` to match `discrete_precision` for all $m$ and $n$.
Expect **coherent** answers to the same actuarial question, not equality.
:::

## What the mode controls

Each mode makes two independent decisions:

1. **Computation method** — exact payment-grid sum, Woolhouse/UDD interpolation from annual
   values, or numerical integration.
2. **Mortality table precision** — whether survival probabilities are derived from the
   *rounded* $l_x$ values stored in the published table, or from the *unrounded*
   floating-point $l_x$ values retained internally during table construction. Rounding is
   configured via `config.decimals.lx` (default 15 decimal places). Discrete modes use
   rounded values — consistent with classical commutation-function practice. Continuous modes
   bypass rounding for higher precision in integration.

## Mode comparison

| Mode | Method | $l_x$ precision | Typical use |
|------|--------|-----------------|-------------|
| `discrete_precision` | Exact payment-grid summation | Rounded | Production (default) |
| `discrete_simplified` | Woolhouse interpolation from annual values | Rounded | Fast estimates, $m>1$ |
| `continuous_precision` | Trapezoidal numerical integration | Unrounded | Highest accuracy |
| `continuous_simplified` | Annual calculations + fractional interpolation | Unrounded | Continuous approximation |

## Mode descriptions

### `discrete_precision` (default)

The most commonly used mode for production calculations.

The engine constructs the exact payment timing grid and evaluates survival probabilities and
discount factors at each payment time, for **all values of $m$** (including $m=1$) and all
starting ages (including fractional).

**Annuities** — direct summation over the payment grid. For an annuity-due
($\ddot{a}$, payments at period start) and an annuity-immediate ($a$, payments at period end):

$$\ddot{a}_{x:\overline{n}|}^{(m)} = \frac{1}{m} \sum_{j=0}^{mn-1} v^{j/m} \cdot {}_{j/m}p_x
\qquad
a_{x:\overline{n}|}^{(m)} = \frac{1}{m} \sum_{j=1}^{mn} v^{j/m} \cdot {}_{j/m}p_x$$

Fractional survival probabilities are computed from the rounded $l_x$ table using the
interpolation method set in `config.lx_interpolation` (default `"linear"`, i.e. the Uniform
Distribution of Deaths (UDD) assumption). This knob does **not** set insurance benefit
timing — see `config.mortality_placement` and {ref}`two-independent-knobs`.

**Insurances** — present value of a benefit payable upon death in each sub-annual period,
discounted to the benefit payment time $\delta_m$ controlled by `config.mortality_placement`
(see {ref}`mortality-placement` below for the offset values):

$$A_{x:\overline{n}|}^{(m)} = \sum_{j=0}^{mn-1} v^{j/m\,+\,\delta_m} \cdot {}_{j/m}p_x \cdot {}_{1/m}q_{x+j/m}$$

Here ${}_{j/m}p_x$ and ${}_{1/m}q_{x+j/m}$ follow `lx_interpolation`; $\delta_m$ follows
`mortality_placement` only. Annuities and pure endowments ignore `mortality_placement`.

**Pure endowments** — exact survival probability and discount factor at the end of the term:

$${}_{n}E_x = v^n \cdot {}_np_x$$

Because this mode constructs the exact payment timing grid, it is the **only** one that
handles **fractional final payments** — applicable to both annuities and insurances. When
$n$ is not an exact multiple of $1/m$: for annuities the last payment amount is
proportionally reduced; for insurances both the death probability of the last sub-annual
period and the benefit timing offset are scaled by the fractional weight. See
{doc}`last_payment_adjustment` for details. `discrete_simplified` and the continuous modes
do not apply this adjustment.

:::{note}
Commutation function formulas such as $\ddot{a}_x = N_x / D_x$ are **not** used internally
by this (or any other) mode. The `Dx()`, `Nx()`, `Cx()`, `Mx()`, etc. methods are standalone
tools for classical calculations, not the internal engine path.
:::

### `discrete_simplified`

Converts from annual-basis values to payment frequency $m$ using the Woolhouse (UDD) 2-term
approximation. This mode requires only two annual calculations regardless of $m$, making it
considerably faster than `discrete_precision` for $m > 1$.

**Annuities-due** — interpolates between the annual annuity-due and annuity-immediate:

$$\ddot{a}_{x:\overline{n}|}^{(m)} \approx \frac{m+1}{2m}\,\ddot{a}_{x:\overline{n}|} + \frac{m-1}{2m}\,a_{x:\overline{n}|}$$

**Annuities-immediate** — the interpolation coefficients are swapped:

$$a_{x:\overline{n}|}^{(m)} \approx \frac{m-1}{2m}\,\ddot{a}_{x:\overline{n}|} + \frac{m+1}{2m}\,a_{x:\overline{n}|}$$

For $m=1$ both formulas reduce to their annual counterpart (no approximation needed).

**Insurances** — linear age interpolation between consecutive annual insurance values:

$$A_x^{(m)} \approx A_x + \frac{m-1}{2m}\bigl(A_{x+1} - A_x\bigr)$$

(Woolhouse applies to **annuities only**; insurances use adjacent-age linear
interpolation — **not** the textbook $(i/i^{(m)})A_x$ UDD frequency adjustment.)

(discrete-simplified-insurance)=
:::{important}
**Insurance `discrete_simplified` is not the textbook formula.**  Actuarial manuals
often write $A_x^{(m)} \approx (i/i^{(m)})\,A_x$ under UDD.  Lactuca instead
interpolates between $A_x$ and $A_{x+1}$ at annual frequency, then applies the
hybrid $k+s$ tail for fractional terms.  Use `discrete_precision` when the exact
$m$-thly benefit grid matters.
:::

**Pure endowments** — applies year-by-year UDD survival to evaluate ${}_np_x$:

$${}_{n}E_x \approx v^n \cdot {}_np_x^{\text{UDD}}$$

For a fractional duration $0 < n \leq 1$: ${}_np_x^{\text{UDD}} = 1 - n\,q_x$ (UDD within the year).
For $n > 1$, each integer year and the terminal fractional year are evaluated under UDD
and multiplied: ${}_np_x^{\text{UDD}} = p_x \cdot p_{x+1} \cdots p_{x+k-1} \cdot (1 - s\,q_{x+k})$,
where $k = \lfloor n \rfloor$ and $s = n - k$.

Slightly less accurate than `discrete_precision` for **integer** terms $n$ and $m > 1$
(typical relative error $< 0.01\%$ for $m \leq 12$). For **fractional** effective terms
$n_\text{eff} \notin \mathbb{Z}$ with $m > 1$, a **hybrid** decomposition applies:

- $k = \lfloor n_\text{eff} \rfloor$ — Woolhouse (annuities) or linear age interpolation
  (insurances) on annual temporaries of duration $k$;
- $s = n_\text{eff} - k$ — exact $m$-thly tail using `discrete_precision` conventions
  (including growth anniversaries at $k + \lfloor j/m \rfloor$).

When $k = 0$ ($n_\text{eff} < 1$), the result **equals** `discrete_precision`. When
$n_\text{eff}$ is an integer, behaviour is unchanged from the pre-hybrid implementation.
Results with $k \geq 1$ **differ** from full `discrete_precision` — that is expected
under mode independence. Use `discrete_precision` when the exact payment grid matters.
Custom `cashflow_times` / `cashflow_amounts` are **not supported** in this mode — use
`discrete_precision` explicitly. Both modes use rounded $l_x$ values.

### `continuous_precision`

Computes present values via numerical integration over unrounded $l_x$ values:

**Annuities:**
$\bar{a}_x = \int_0^\infty e^{-\delta t} \cdot {}_t p_x\, \mathrm{d}t$

**Insurances:**
$\bar{A}_x = \int_0^\infty e^{-\delta t} \cdot {}_t p_x \cdot \mu_{x+t}\, \mathrm{d}t$

**Pure endowments:**
$${}_{n}E_x = \exp\!\left(-\int_0^n \bigl[\delta(t) + \mu_{x+t}\bigr]\,\mathrm{d}t\right)$$

The integrals are evaluated with the **trapezoidal rule** over a fine uniform grid.
Default resolution differs by product type: **annuities** use 200 steps per unit of time
(total steps $= \lceil 200n \rceil$, scaling with term length);
**insurances** use exactly 1000 integration points;
**pure endowments** use $\max(1000,\, 50n)$ points — ensuring at least 50 steps per year
for long-duration contracts (the extra resolution kicks in when $n > 20$ years).
All survival probability evaluations use unrounded $l_x$ values, giving the highest precision.

### `continuous_simplified`

Avoids numerical integration by combining exact annual calculations with actuarial
interpolation. Like `continuous_precision`, unrounded $l_x$ values are used throughout.

**Annuities** — for term $n$ with integer part $k = \lfloor n \rfloor$ and fractional part
$s = n - k$, the implementation applies a composite trapezoidal rule with step 1 for the
integer portion and a single trapezoidal step of width $s$ for the terminal fractional period:

$$\bar{a}_{x:\overline{n}|} \approx \frac{\ddot{a}_{x:\overline{k}|} + a_{x:\overline{k}|}}{2} + \frac{s}{2}\!\left(v^k\,{}_kp_x + v^n\,{}_np_x\right)$$

The first term — average of the $m=1$ due and immediate annuities — is the
composite trapezoidal approximation of $\bar{a}_{x:\overline{k}|}$. The second
term is a one-step trapezoid over $[k,\,n]$. Only two annual annuity
calculations are needed regardless of $k$.

:::{note}
The annuity formula above is the **composite trapezoidal rule** with step $h = 1$.
Its approximation error relative to the exact continuous integral scales as $O(h^2)$,
the same order as the standard trapezoidal rule. For higher precision, switch to
`continuous_precision`, which evaluates the integral on a fine grid (default 200 steps
per year) and achieves correspondingly smaller errors.
:::

**Insurances** — the value is the arithmetic mean of two `continuous_precision`
evaluations at ages $x$ and $x+1$. Denoting the precision result at age $y$ as
$\bar{A}_y^{\text{prec}}$, the simplified result is:

$$\bar{A}_x^{(\text{simp})} = \tfrac{1}{2}\!\left(\bar{A}_x^{\text{prec}} + \bar{A}_{x+1}^{\text{prec}}\right)$$

This formula applies for all starting ages (integer or fractional).

**Pure endowments** — uses the average force of mortality and interest:

$${}_{n}E_x \approx \exp\!\left[-n\!\left(\bar{\delta} + \bar{\mu}_x\right)\right]$$

where $\bar{\delta} = -\ln v^n / n$ and $\bar{\mu}_x = -\ln({}_np_x) / n$ are the average
forces of interest and mortality over the term. This is considerably faster than numerical
integration while remaining accurate for slowly varying mortality and interest rates.

## Setting the mode

```python
from lactuca import config

config.calculation_mode = "discrete_precision"    # default
config.calculation_mode = "discrete_simplified"
config.calculation_mode = "continuous_precision"
config.calculation_mode = "continuous_simplified"
```

Or in a TOML config file:

```toml
[calculation]
calculation_mode = "continuous_precision"
```

For a full reference of all configuration settings, see {doc}`configuration`.

## Payment frequencies

The `m` parameter (payments per year) applies to all four modes and is restricted to these
values:

| `m` | Payments per year |
|-----|-------------------|
| `1` | Annual |
| `2` | Semi-annual |
| `3` | Every four months |
| `4` | Quarterly |
| `6` | Bi-monthly |
| `12` | Monthly |
| `14` | 14 payrolls/year — Spanish "14 pagas" scheme (**approximate**, see note below) |
| `24` | Twice-monthly (equispaced under 360-day commercial year) |
| `26` | Biweekly (equispaced under 364-day year: 52×7 days) |
| `52` | Weekly |
| `365` | Daily |

Passing any other value raises `ValueError`.

:::{warning}
`m=14` is accepted for convenience but is **not equispaced** under any standard year
convention (365÷14≈26.07 days; 360÷14≈25.71 days). Using it in the standard fractional
annuity formula introduces a systematic approximation error. For actuarially precise
results, model the benefit as a monthly annuity (`m=12`) plus two extraordinary
cash flows at mid-year and year-end using {doc}`irregular_cashflows`.
:::


(two-independent-knobs)=
## Two independent mortality settings

Lactuca exposes **two independent** global settings. Both involve *when death occurs* in
actuarial language, but they control **different calculation steps**. Changing one does
**not** substitute for the other.

| Setting | Controls | Used in | Does **not** affect |
|---------|----------|---------|---------------------|
| `config.lx_interpolation` | Bridging integer-age $l_x$ to fractional ages — UDD (`"linear"`) or constant force (`"exponential"`) | ${}_{j/m}p_x$, ${}_{1/m}q_x$, annuity payment grids, pure endowment ${}_np_x$, continuous-mode integration | Insurance discount offset $\delta_m$; classical identity $1 = d\,\ddot{a}_x + A_x$ |
| `config.mortality_placement` | Timing of the death **benefit payment** within each sub-annual period ($\delta_m$ in insurance discount) | **Insurances** (`Ax`) only; commutation $C_x$ exponent | Annuities ($\ddot{a}_x$, $a_x$); pure endowments (${}_n E_x$); how ${}_{j/m}p_x$ is computed |

In `discrete_precision` insurance, both settings compose: survival probabilities and
interval death probabilities come from `lx_interpolation`; the present-value exponent
adds $\delta_m$ from `mortality_placement`. See {doc}`lx_interpolation` and
{ref}`mortality-placement` below.

(mortality-placement)=
## Insurance: mortality placement

`config.mortality_placement` controls when within each sub-annual period the death benefit
is assumed to be paid. It affects **insurances only** — annuities and pure endowments are
not affected. This setting is **independent** of `config.lx_interpolation` (fractional-age
survival); see {ref}`two-independent-knobs`.

| `mortality_placement` | $\delta_m$ | Time offset within period | Convention |
|-----------------------|------------|--------------------------|------------|
| `"beginning"` | $0$ | Start of period | Payment immediately on death |
| `"mid"` | $\dfrac{1}{2m}$ | Mid-period | Mid-sub-period payment (default) |
| `"end"` | $\dfrac{1}{m}$ | End of period | Traditional commutation-function convention |

```python
from lactuca import config

config.mortality_placement = "mid"       # default
config.mortality_placement = "end"       # traditional end-of-year convention
config.mortality_placement = "beginning" # payment immediately on death
```

:::{warning}
**Default `"mid"` breaks the textbook identity.**  Classical commutation algebra gives
$1 = d\,\ddot{a}_x + A_x$ only when benefits are discounted at **end of year of death**
(`mortality_placement = "end"`) and $m = 1$.  With the library default `"mid"`,
$C_x = v^{x+1/2} d_x$ and the identity does **not** close numerically.  This mid-period
benefit timing is intentional and **unrelated** to `lx_interpolation` (UDD vs CFM); set
`"end"` when you need textbook duality.
:::

## Accuracy comparison

See {ref}`actuarial-coherence-of-modes` — modes are actuarially coherent but not numerically
identical. The table below summarizes **typical** numerical closeness; it is not a guarantee
of equality.

For most regulatory and commercial valuations, `discrete_precision` with $m \leq 12$ is often
**close** to `continuous_precision` (within rounding for standard curves). Differences become
more visible at:

- Very high payment frequency ($m = 52$ weekly or $m = 365$ daily).
- Very small interest rates ($i < 0.5\%$) where the frequency adjustment is relatively large.
- Extreme ages ($x > 90$) where the mortality gradient is steep.

`discrete_simplified` vs. `discrete_precision` differences are typically below 0.1% for
$m \leq 12$, **integer** terms, and standard mortality curves. For **fractional** $n$ with
$m > 1$, simplified uses the hybrid $k+s$ split above; annuities are typically within
0.1% of precision, while insurances may differ by a few percent on the integer-year
approximation. The Woolhouse approximation degrades slightly for very high $m$
or rapidly varying mortality schedules.

(classical-identity)=
## Classical identity $1 = d\,\ddot{a}_x + A_x$

The textbook whole-life identity holds only when **both** conditions are met:

1. `m = 1` (annual payments), and
2. `config.mortality_placement = "end"` (benefit at end of year of death).

Changing `config.lx_interpolation` (`"linear"` vs `"exponential"`) does **not** restore
this identity — that setting governs fractional survival, not insurance benefit timing.

The library default is `mortality_placement = "mid"`, so $C_x = v^{x+1/2} d_x$ in commutation
functions and engine insurances use a mid-period discount offset. Do **not** expect
$1 = d\,\ddot{a}_x + A_x$ numerically under default settings.

(regulatory-reporting-scope)=
## Regulatory reporting scope

Lactuca computes **actuarial present values** and related quantities. Whether those outputs
map to a regulatory liability depends on **your inputs and reporting framework** — the library
does not certify compliance with any standard.

| Use case | Supported as a calculation building block | Qualifiers |
|---|---|---|
| Solvency II technical provisions (BEL component) | Yes | User supplies mortality, decrements, and discount assumptions. Default `InterestRate` is a **flat** annual rate unless you provide a term structure manually — see {doc}`interest_rates_guide`. EIOPA RFR has no official loader — see {doc}`../formulas`. |
| IFRS 17 fulfilment cash flows (BEL / expected CF component) | Same as BEL | PV machinery only — not measurement model choice (GMM vs PAA), contract boundary, or discounting methodology beyond user inputs |
| IFRS 17 risk adjustment (RA), CSM, loss component | **Out of scope** | User responsibility outside Lactuca |
| Solvency II SCR / internal model | **Out of scope** | No stressed scenarios or correlation engine |

For BEL-style work, `discrete_precision` with $m \leq 12$ is the usual discrete reference
(see {ref}`actuarial-coherence-of-modes` and **Accuracy comparison** above). Continuous modes
are also valid when consistent with your valuation basis.

## See also

- {doc}`modifying_decrements` — `table_combination`, `combination_mode` (`independent` / `udd`), and decrement adjustments
- {doc}`inspecting_cashflows` — `return_flows=True` parameter: full dict structures returned by each mode and product type (annuity, insurance, pure endowment)
- {doc}`lx_interpolation` — fractional-age survival probabilities under UDD and CFM
- {doc}`force_mortality_methods` — force of mortality methods and `mortality_placement`
- {doc}`prospective_reserve` — fractional policy year (`ts` parameter)
- {doc}`numerical_precision` — rounded vs. unrounded $l_x$ in detail
- {doc}`batch_calculations` — performance notes: `discrete_precision` 50–250× vs all other modes ~1.5× speedup

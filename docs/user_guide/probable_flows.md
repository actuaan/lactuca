# Probable Flows and Portfolio Reserves

**Probable flows** are the **expected cash flows** of a portfolio of insurance or
pension contracts: the benefit amounts weighted by the probability that each benefit
is actually triggered at each future point in time.  Their discounted sum is the
actuarial present value (APV) of the portfolio — the **Best Estimate Liability (BEL)**
under IFRS 17 and Solvency II, or the **Present Value of the Defined Benefit Obligation
(PVDBO)** under IAS 19.

This guide covers the theory, the Lactuca API, and complete worked examples for:

- IFRS 17 Best Estimate Liability (life insurance contracts)
- Solvency II mortality and longevity stress tests (SCR life module)
- IAS 19 Present Value of the Defined Benefit Obligation (pension funds)
- Sensitivity analysis — interest-rate duration and mortality sensitivity

```{seealso}
{doc}`batch_calculations` — Quick reference for `return_flows=True` and the
portfolio-flow dict keys.\
{doc}`inspecting_cashflows` — Single-policy cash-flow inspection.\
{doc}`prospective_reserve` — Prospective reserve and the `ts` parameter.\
{doc}`interest_rates_guide` — `InterestRate` construction and term-structure input.
```

---

## Theory: from individual APV to portfolio expected flows

### Single-policy annuity

For a single life aged $x$ receiving a unit annuity-immediate (postpagable) with
$m$ payments per year over $n$ years, the payment at time $t_j = j/m$
($j = 1, 2, \ldots, mn$) occurs with probability ${}_{t_j}p_x$.
The **actuarial present value** is:

$$a_{x:\overline{n}|}^{(m)} = \frac{1}{m}\sum_{j=1}^{mn} v^{t_j}\,{}_{t_j}p_x$$

The **undiscounted expected cash flow** at $t_j$ is simply:

$$\mathbb{E}[\text{CF}_{t_j}] = \frac{1}{m}\,{}_{t_j}p_x$$

Summing over $j$ gives the expected total number of payments;
weighting by $v^{t_j}$ gives $a_{x:\overline{n}|}^{(m)}$.

### Single-policy insurance

For a term insurance with sum insured $S$ payable at the end of the period of death,
the expected cash flow at interval $j$ is:

$$\mathbb{E}[\text{CF}_{t_j}] = S \cdot {}_{t_{j-1}}p_x \cdot q_{x+t_{j-1}}^{(1/m)}$$

where $q_{x+s}^{(1/m)}$ is the mortality probability over one sub-period.
The APV is $A_{x:\overline{n}|}^{1} = \sum_j v^{t_j^*} \cdot \mathbb{E}[\text{CF}_{t_j}]$,
where $t_j^*$ is the discounting time within the interval.

### Portfolio aggregation

For a portfolio of $K$ policies with ages $x_1, \ldots, x_K$ and
benefit amounts $S_1, \ldots, S_K$, the aggregate expected cash flow at time $t_j$ is:

$$\mathbb{E}[\text{CF}_{t_j}^{\text{portfolio}}]
  = \sum_{k=1}^{K} S_k \cdot {}_{t_j}p_{x_k}^{(k)}$$

where ${}_{t_j}p_{x_k}^{(k)}$ denotes the survival probability from the table of
the $k$-th policy.  The total APV (BEL at technical rate) is:

$$\text{APV} = \sum_j v^{t_j} \cdot \mathbb{E}[\text{CF}_{t_j}^{\text{portfolio}}]$$

Lactuca computes this sum **without any Python loop over policies**:
the batch engine accumulates contributions on a shared time grid using NumPy
scatter-add operations.

---

## What `return_flows=True` returns in batch mode

When `return_flows=True` is passed with an array of ages the method returns a
**portfolio-level flow dict** instead of a scalar.

This aggregate-flow path requires `config.calculation_mode` in
`'discrete_precision'` or `'continuous_precision'`. In simplified modes,
batch calls with `return_flows=True` raise `ValueError`.

| Key | Type | Content |
|---|---|---|
| `time_grid` | `NDArray[float64]` | Union of all policy payment times, sorted ascending |
| `expected_cf` | `NDArray[float64]` | Undiscounted aggregate expected cash flows at each grid point |
| `pv_cf` | `NDArray[float64]` | Discounted contributions: `expected_cf * discount_factor` |
| `total_pv` | `float` | `np.sum(pv_cf)` — total APV / BEL / PVDBO |

The **separation of `expected_cf` and `pv_cf`** is the key design decision: because
undiscounted flows are stored independently, you can apply *any* discount curve after
the fact — the EIOPA risk-free curve, a corporate bond yield, a government curve with
VA/MA adjustment — without re-running the mortality calculation.

---

## IFRS 17 Best Estimate Liability

Under IFRS 17, the **Best Estimate Liability** is the present value of all future cash
outflows under the insurance contracts, discounted at the IFRS 17 discount curve
(EIOPA risk-free rate plus any applicable Volatility Adjustment).

$$\text{BEL} = \sum_j v_{\text{EIOPA}}(t_j) \cdot \mathbb{E}[\text{CF}_{t_j}^{\text{portfolio}}]$$

The discount curve is external to the mortality model — Lactuca separates them cleanly.

### Worked example: term life insurance portfolio

```python
from lactuca import LifeTable, InterestRate

# ── Portfolio data (50 policies) ─────────────────────────────────────────────
ages  = [35, 40, 42, 38, 55, 60, 45, 50, 48, 52,
         36, 41, 43, 39, 56, 61, 46, 51, 49, 53,
         37, 42, 44, 40, 57, 62, 47, 52, 50, 54,
         35, 40, 42, 38, 55, 60, 45, 50, 48, 52,
         36, 41, 43, 39, 56, 61, 46, 51, 49, 53]

# 20-year term for all policies; uniform sum insured for this illustration
sum_insured = 100_000.0   # EUR — apply as a scaling factor below

# ── Table at technical rate ──────────────────────────────────────────────────
lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)

# ── Step 1: obtain undiscounted expected flows (unit capital = 1 per policy) ─
# For heterogeneous per-policy sums insured use benefits= (see note below).
# This example scales the aggregate expected_cf by a uniform sum_insured afterwards.
flows = lt.Ax(ages, n=20, return_flows=True)

t_grid      = flows["time_grid"]                      # payment times on the union grid
expected_cf = flows["expected_cf"] * sum_insured   # scale: unit flows × capital

# ── Step 2: apply EIOPA risk-free discount curve ─────────────────────────────
# (Replace 0.0342 with the published flat-rate approximation or a term-structure)
eiopa_rate  = InterestRate(0.0342)
v_eiopa     = eiopa_rate.vn(t_grid)     # discount factors at each t_grid point

bel         = expected_cf.dot(v_eiopa)
print(f"BEL (EIOPA @ 3.42%): {bel:,.2f}")   # illustrative

# ── Step 3: BEL at technical rate (for comparison) ──────────────────────────
print(f"APV at technical rate (3.00%): {flows['total_pv'] * sum_insured:,.2f}")
```

:::{note}
**Per-policy sums insured**

The primary API for per-policy benefit amounts is `benefits=`:

```python
sums = [100_000, 150_000, 200_000, 80_000]
flows = lt.Ax(ages, n=20, ir=eiopa_rate,
              return_flows=True, benefits=sums)
bel = flows["total_pv"]   # already weighted and discounted
```

`benefits=` scales each policy's contribution before aggregation; the resulting
`expected_cf` and `total_pv` already reflect the per-policy amounts.

For scalar BEL without per-period flows, the direct alternative is unit APV dotted
with the benefit vector:

```python
unit_apv = lt.Ax(ages, n=20, ir=eiopa_rate)   # NDArray shape (N,)
sums = [100_000, 150_000, 200_000, 80_000]
bel = unit_apv @ sums
```
:::

:::{note}
The two BEL values differ because the discount rate differs.  For IFRS 17 reporting,
always apply the **EIOPA curve** (`eiopa_rate.vn(t_grid)`) rather than the pricing
rate already embedded in the table.  The pricing rate is implicit in `total_pv`; the
IFRS 17 BEL requires step 2 above.
:::

### Term-structure discount curve

For a full term structure (non-flat EIOPA curve), pass segment lengths and per-segment
rates to `InterestRate`:

The variables `t_grid` and `expected_cf` are defined in the worked example above.

```python
from lactuca import LifeTable, InterestRate

# ── Reproduce the shared setup ────────────────────────────────────────────────
ages = [35, 40, 42, 38, 55, 60, 45, 50, 48, 52,
        36, 41, 43, 39, 56, 61, 46, 51, 49, 53,
        37, 42, 44, 40, 57, 62, 47, 52, 50, 54,
        35, 40, 42, 38, 55, 60, 45, 50, 48, 52,
        36, 41, 43, 39, 56, 61, 46, 51, 49, 53]
sum_insured = 100_000.0
lt          = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)
flows       = lt.Ax(ages, n=20, return_flows=True)
t_grid      = flows["time_grid"]
expected_cf = flows["expected_cf"] * sum_insured

# ── EIOPA forward segment rates (illustrative — replace with published values) ─
# terms = duration of each segment (years); rates has len(terms)+1 entries (last = perpetuity)
term_lengths   = [0.5, 0.5, 1.0, 1.0, 2.0, 2.0, 3.0, 5.0, 5.0, 10.0]
segment_rates  = [0.031, 0.033, 0.034, 0.035, 0.036, 0.036,
                  0.035, 0.034, 0.033, 0.032, 0.032]

eiopa_curve = InterestRate(terms=term_lengths, rates=segment_rates)
v_eiopa     = eiopa_curve.vn(t_grid)

bel_curve   = expected_cf.dot(v_eiopa)
print(f"BEL (EIOPA term structure): {bel_curve:,.2f}")
```

:::{seealso}
{doc}`../api/interest_rate` — Full `InterestRate` reference including spot-rate
conversion, multi-scenario containers, and the `vn()` method.
:::

### Regulatory monthly grid (`t_output`)

IFRS 17 actuarial models sometimes require cash flows bucketed on a fixed monthly
grid for reconciliation with the finance system.  `t_output` requires `return_flows=True`
in batch mode; passing `t_output` without `return_flows=True` raises a `ValueError`, and
passing `t_output` in scalar mode also raises a `ValueError`.  In batch mode,
`return_flows=True` requires `config.calculation_mode` in `'discrete_precision'` or
`'continuous_precision'`. Pass `t_output` to force this bucketing:

The variables `lt`, `ages`, `sum_insured`, and `eiopa_rate` are defined in the
worked example above.

```python
from lactuca import payment_times

# Fixed monthly grid: 1/12, 2/12, ..., 20 (20-year, monthly)
reg_grid = payment_times(n=20, m=12)

flows_monthly = lt.Ax(ages, n=20, ir=eiopa_rate, t_output=reg_grid, return_flows=True)

# total_pv is always exact (payment times are not bucketed for PV computation)
bel_monthly = flows_monthly["total_pv"] * sum_insured

# expected_cf gives the bucketed flows for period-by-period reporting / roll-forward
monthly_ecf = flows_monthly["expected_cf"] * sum_insured
```

:::{warning}
**Timing approximation**

When `t_output` is provided, each payment is assigned to the nearest earlier grid
point via a bucket-accumulation algorithm.  This introduces a small timing error
(typically sub-basis-point for monthly grids) that grows with coarser grids.
Use the **exact union grid** (`t_output=None`, the default) for auditable BEL
reconciliations; use `t_output` only when a fixed reporting grid is explicitly
required by the model specification.
:::

:::{seealso}
{ref}`payment-time-grid-options` in {doc}`batch_calculations` — complete bucketing
mechanics, three grid-regime analysis (coarser / finer / gapped), IFRS 17
re-discounting guidance, and the guarantee that `total_pv` is always exact.
:::

:::{tip}
When a portfolio contains multiple product types (e.g. term insurance + pension
annuity), applying the same `t_output=` grid to each batch call produces
`expected_cf` arrays on a common time axis.  They can then be added element-wise
and discounted once — one discount pass for the whole portfolio.

Requires `return_flows=True` in `'discrete_precision'` or `'continuous_precision'`.
:::

---

## Solvency II SCR — life underwriting risk module

The Solvency Capital Requirement (SCR) for life underwriting risk in Solvency II is
computed as the **Value-at-Risk at 99.5% confidence over one year**, approximated using
the standard formula by applying prescribed stress scenarios to the best estimate.

### Mortality stress (mortality risk sub-module)

For insurance products where higher mortality increases the liability
(term life, whole life), the Solvency II mortality stress is a **permanent 15% increase**
in all $q_x$ rates ($q_x \to 1.15 \cdot q_x$, capped at 1):

$$\text{SCR}_{\text{mortality}} \approx
\max\!\left(\text{BEL}^{\text{stress}} - \text{BEL}^{\text{base}},\,0\right)$$

```python
from lactuca import LifeTable, InterestRate

# ── Shared portfolio data (same as IFRS 17 worked example above) ─────────────
ages = [35, 40, 42, 38, 55, 60, 45, 50, 48, 52,
        36, 41, 43, 39, 56, 61, 46, 51, 49, 53,
        37, 42, 44, 40, 57, 62, 47, 52, 50, 54,
        35, 40, 42, 38, 55, 60, 45, 50, 48, 52,
        36, 41, 43, 39, 56, 61, 46, 51, 49, 53]
sum_insured = 100_000.0

lt_base   = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)
lt_stress = lt_base.copy()
lt_stress.modify_qx({"decrement_multiplier": 1.15})  # +15% qx (Solvency II mortality stress)

eiopa_rate = InterestRate(0.0342)

bel_base   = lt_base.Ax(ages, n=20, ir=eiopa_rate).sum() * sum_insured
bel_stress = lt_stress.Ax(ages, n=20, ir=eiopa_rate).sum() * sum_insured

scr_mortality = max(bel_stress - bel_base, 0.0)
print(f"BEL base:     {bel_base:,.2f}")
print(f"BEL stressed: {bel_stress:,.2f}")
print(f"SCR mortality: {scr_mortality:,.2f}")
```

### Longevity stress (longevity risk sub-module)

For products where **lower mortality increases the liability** (annuities, pensions),
the Solvency II longevity stress is a **permanent 20% decrease** in all $q_x$ rates:

$$q_x^{\text{stress}} = 0.80 \cdot q_x$$

$$\text{SCR}_{\text{longevity}} \approx
\max\!\left(\text{APV}^{\text{stress}} - \text{APV}^{\text{base}},\,0\right)$$

```python
from lactuca import LifeTable, InterestRate

# ── Shared portfolio data (same ages as mortality stress example above) ───────
ages = [35, 40, 42, 38, 55, 60, 45, 50, 48, 52,
        36, 41, 43, 39, 56, 61, 46, 51, 49, 53,
        37, 42, 44, 40, 57, 62, 47, 52, 50, 54,
        35, 40, 42, 38, 55, 60, 45, 50, 48, 52,
        36, 41, 43, 39, 56, 61, 46, 51, 49, 53]
eiopa_rate = InterestRate(0.0342)

lt_long_base   = LifeTable("PER2020_Ind_1o", "m")
lt_long_stress = lt_long_base.copy()
lt_long_stress.modify_qx({"decrement_multiplier": 0.80})  # -20% qx (Solvency II longevity stress)

apv_base   = lt_long_base.ax(ages, n=20, ir=eiopa_rate).sum()
apv_stress = lt_long_stress.ax(ages, n=20, ir=eiopa_rate).sum()

scr_longevity = max(apv_stress - apv_base, 0.0)
print(f"APV base:     {apv_base:,.2f}")
print(f"APV stressed: {apv_stress:,.2f}")
print(f"SCR longevity: {scr_longevity:,.2f}")
```

:::{note}
`modify_qx` modifies only the copied table; `lt_long_base` is unchanged.
See {doc}`modifying_decrements` for the full `modify_qx` reference.
:::

### Lapse risk stress (mass-lapse sub-module)

Lapse risk is out of scope for a pure life-table model; it requires a surrender-value
model.  When lapse probabilities are folded into a multiple-decrement table, the standard
batch API (passing `ir=` directly) already produces lapse-adjusted present values without
any extra steps.  Use `return_flows=True` only if you additionally need the temporal
cashflow profile for disclosure purposes
(see {doc}`modifying_decrements`).

---

## IAS 19 Present Value of the Defined Benefit Obligation

Under IAS 19, the PVDBO is discounted at **high-quality corporate bond yields** (or
government bond yields in thin corporate bond markets).  The structure is identical
to the BEL calculation; only the discount rate changes.

### Worked example: pension annuity portfolio

```python
from lactuca import LifeTable, InterestRate

# Current retirees: ages at valuation date, annual pension amounts
pensioner_ages = [65, 67, 70, 72, 75, 68, 63, 71, 74, 66]
annual_pension = [24_000.0, 18_000.0, 30_000.0, 22_000.0, 15_000.0,
                  20_000.0, 28_000.0, 17_000.0, 12_000.0, 25_000.0]

lt = LifeTable("PASEM2010", "m", interest_rate=0.03)  # static table — no cohort required
ia19_rate = InterestRate(0.0380)

# PVDBO: per-pensioner APV × annual pension, discounted at IAS 19 rate (monthly, m=12, postpagable)
pvdbo = lt.ax(pensioner_ages, m=12, ir=ia19_rate, benefits=annual_pension).sum()

# APV at technical rate for comparison
apv_technical = lt.ax(pensioner_ages, m=12, benefits=annual_pension).sum()

print(f"PVDBO (IAS 19 rate 3.80%):     {pvdbo:,.2f}")
print(f"APV at technical rate (3.00%): {apv_technical:,.2f}")
```

:::{note}
**Generational tables for pension plans**

Long-duration pension plans should use a **generational table** (e.g.
`PER2020_Ind_1o` with `cohort=` equal to the birth year of each member) to
reflect future mortality improvements.  When birth years differ across the
membership, use the multi-table batch pattern (lookup dict by cohort) described
in {doc}`batch_calculations`.
:::

:::{tip}
**Multi-scenario evaluation (option B)**

When the same portfolio must be valued at several discount rates (e.g. technical
rate and IAS 19 rate simultaneously), computing the mortality cashflows once with
`return_flows=True` and re-discounting with `vn()` is more efficient than
repeating the full batch call.  The mortality model runs once; each additional
rate scenario costs only $O(T)$ discount-factor evaluations ($T$ = time-grid
length, typically 120–240 for 10–20-year terms):

```python
tech_rate = InterestRate(0.03)

flows = lt.ax(pensioner_ages, m=12, return_flows=True, benefits=annual_pension)
ecf, tg = flows["expected_cf"], flows["time_grid"]

pvdbo         = ecf.dot(ia19_rate.vn(tg))
apv_technical = ecf.dot(tech_rate.vn(tg))
```

Requires `discrete_precision` or `continuous_precision` calculation mode
(raises `ValueError` in simplified modes).
:::

---

## Sensitivity analysis

### Interest-rate sensitivity (modified duration)

The **modified duration** of a portfolio's BEL measures the sensitivity to a
parallel shift $\Delta r$ in the discount curve:

$$D_{\text{mod}} = -\frac{1}{\text{BEL}} \cdot \frac{\partial \text{BEL}}{\partial r}
  \approx \frac{\text{BEL}(r - \Delta r) - \text{BEL}(r + \Delta r)}{2\,\Delta r\,\text{BEL}}$$

Because `expected_cf` is independent of the discount rate, computing the bumped BEL
requires only re-evaluating the discount factors — not re-running the mortality model:

```python
from lactuca import LifeTable, InterestRate

ages = [35, 40, 42, 38, 55, 60, 45, 50, 48, 52,
        36, 41, 43, 39, 56, 61, 46, 51, 49, 53,
        37, 42, 44, 40, 57, 62, 47, 52, 50, 54,
        35, 40, 42, 38, 55, 60, 45, 50, 48, 52,
        36, 41, 43, 39, 56, 61, 46, 51, 49, 53]
sum_insured = 100_000.0

lt    = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)
flows = lt.Ax(ages, n=20, return_flows=True)

ecf    = flows["expected_cf"] * sum_insured   # scaled expected cash flows
t_grid = flows["time_grid"]

r0 = 0.0342
dr = 0.0001   # 1 basis point

bel_mid  = ecf.dot(InterestRate(r0       ).vn(t_grid))
bel_up   = ecf.dot(InterestRate(r0 + dr  ).vn(t_grid))
bel_down = ecf.dot(InterestRate(r0 - dr  ).vn(t_grid))

dv01     = bel_down - bel_up       # DV01: change in BEL per +1bp move in rates
dur_mod  = (bel_down - bel_up) / (2.0 * dr * bel_mid)

print(f"BEL at {r0:.2%}:  {bel_mid:,.2f}")
print(f"DV01 (+1bp):       {dv01:+,.2f}")
print(f"Modified duration: {dur_mod:.4f} years")
```

The key efficiency: `expected_cf` is computed once from the mortality model;
the full rate sensitivity grid is obtained by looping only over cheap
`InterestRate(r).vn(t_grid)` evaluations.

### Full rate curve: multi-scenario BEL

The variables `ecf` and `t_grid` are defined in the interest-rate sensitivity block
immediately above.

```python
scenarios = np.linspace(0.01, 0.06, 51)   # 1% to 6% in 10bp steps

bel_by_rate = np.array([
    ecf.dot(InterestRate(r).vn(t_grid))
    for r in scenarios
])
# bel_by_rate[i] is the BEL at scenarios[i]
```

### Mortality sensitivity

To quantify the impact of a 1% parallel shift in $q_x$:

```python
from lactuca import LifeTable, InterestRate

ages = [35, 40, 42, 38, 55, 60, 45, 50, 48, 52,
        36, 41, 43, 39, 56, 61, 46, 51, 49, 53,
        37, 42, 44, 40, 57, 62, 47, 52, 50, 54,
        35, 40, 42, 38, 55, 60, 45, 50, 48, 52,
        36, 41, 43, 39, 56, 61, 46, 51, 49, 53]
sum_insured = 100_000.0

lt_base  = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)
lt_shock = lt_base.copy()
lt_shock.modify_qx({"decrement_multiplier": 1.01})  # +1% qx sensitivity

eiopa = InterestRate(0.0342)

bel_base  = lt_base.Ax(ages, n=20, ir=eiopa).sum() * sum_insured
bel_shock = lt_shock.Ax(ages, n=20, ir=eiopa).sum() * sum_insured

delta_bel = bel_shock - bel_base
print(f"BEL sensitivity to +1% qx: {delta_bel:+,.2f}")
```

---

## Performance notes

| Operation | Cost | Notes |
|---|---|---|
| Standard batch call `Ax` / `ax` with `ir=` | $O(K \cdot mn)$ | Mortality accumulation + discounting in one pass |
| Multi-rate batch call `ir=[r1, r2, …]` | $O(K \cdot mn \cdot S)$ | $S$ = number of rate scenarios |
| `return_flows=True` + re-discount `ecf.dot(vn(tg))` | $O(K \cdot mn) + S \cdot O(T)$ | Mortality run once; each extra scenario costs only $O(T)$ |
| Solvency II stress (2 tables) | $2 \times O(K \cdot mn)$ | Two independent batch calls |

For a monthly portfolio of 100 000 policies with 20-year terms ($T \approx 240$,
$K = 100\,000$, $mn = 240$): a standard batch call costs $O(2.4 \times 10^7)$
NumPy operations.  Re-discounting an already-computed `expected_cf` array costs
only $O(240)$ — roughly $10^5$ times cheaper.

The `return_flows=True` pattern therefore pays off when the **same portfolio** must
be priced at many discount rates (e.g. 50-point rate curve, IAS 19 vs technical
rate, IFRS 17 CSM unlock scenarios): compute mortality once, then loop over
`ecf.dot(InterestRate(r).vn(tg))` at negligible marginal cost.  For one or two
rates, two independent batch calls with `ir=` are simpler and equally fast.

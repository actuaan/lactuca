# Inspecting Cash Flows

The `return_flows=True` parameter causes any calculation method to return not just a scalar
present value but a **full dict of the arrays that produced it**.  This is the primary
mechanism for regulatory traceability, BEL reconciliation, and debugging complex calculations
under IFRS 17 and Solvency II.

```python
from lactuca import LifeTable

lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)
result = lt.äx(65, n=20, m=12, return_flows=True)

# result is a dict — inspect all available keys:
print(list(result.keys()))
# ['payment_time', 'interest_rate', 'discount_factor', 'survival_probability',
#  'growth', 'payment_adjustment', 'present_value_raw', 'present_value']

# Reconstruct the scalar PV:
pv = result["present_value"].sum()
print(round(pv, 4))  # 13.2805
```

The structure of the returned dict depends on the **calculation mode** in effect:

| Active mode | Dict structure |
|-------------|----------------|
| `discrete_precision` (default) | Flat arrays — see [Annuity keys](#annuity-cash-flow-keys-discrete-precision), [Insurance keys](#insurance-cash-flow-keys-discrete-precision), [Pure endowment](#pure-endowment-cash-flow-keys) |
| `continuous_precision` | Integration-grid arrays — see [Continuous mode keys](#continuous-mode-keys) |
| `discrete_simplified` / `continuous_simplified` | Nested dicts (annuity/insurance); flat scalars (endowment) — see [Simplified modes](#simplified-modes-nested-structure) |

---

(annuity-cash-flow-keys-discrete-precision)=
## Annuity cash-flow keys (`discrete_precision`)

All arrays contain one entry per payment interval.
For $m$ payments per year over $n$ years that is $m \times n$ entries
(e.g. $12 \times 20 = 240$ for the example below).

| Key | Description |
|-----|-------------|
| `payment_time` | Payment times $t_j = j/m + d$, $j = 0, 1, \ldots, mn-1$ |
| `interest_rate` | Effective annual rates at each $t_j$ (piecewise-term aware) |
| `discount_factor` | Discount factors $v^{t_j}$ |
| `survival_probability` | Survival probabilities ${}_{t_j}p_x$ |
| `growth` | Benefit growth factors at $t_j$ (1.0 if no `gr=`); key is `"amount"` instead when `cashflow_amounts` is provided |
| `payment_adjustment` | Fractional-final-payment scale (1.0 for full payments; < 1 for the last fractional period) |
| `present_value_raw` | ${}_{t_j}p_x \cdot v^{t_j} \cdot g(t_j) \cdot \text{adj}_j$ — element-wise product before frequency scaling |
| `present_value` | Contribution of each payment to the total PV: $\text{raw}_j / m$ |

The total annuity value is `result["present_value"].sum()`.

---

(insurance-cash-flow-keys-discrete-precision)=
## Insurance cash-flow keys (`discrete_precision`)

All arrays contain one entry per payment interval — same length semantics as the annuity
table above ($m \times n$ entries total).

| Key | Description |
|-----|-------------|
| `payment_index` | Integer interval indices $j = 0, 1, \ldots$ |
| `payment_time` | Policy-year starts for each interval: $t_j = j/m + d$ |
| `discount_time` | Times at which the benefit is discounted: $t_j +$ `mortality_placement / m` |
| `interest_rate` | Effective annual rates at `discount_time` |
| `discount_factor` | Discount factors $v^{\text{discount\_time}}$ |
| `death_probability_raw` | Raw per-interval death probabilities from the table |
| `death_probability_adjustment` | Fractional-final-period scale |
| `death_probability` | Adjusted death probabilities used in the summation |
| `growth` | Benefit growth factors (1.0 if no `gr=`); key is `"amount"` instead when `cashflow_amounts` is provided |
| `present_value` | PV contribution of each interval |

The total insurance value is `result["present_value"].sum()`.

---

(pure-endowment-cash-flow-keys)=
## Pure-endowment cash-flow keys

Because a pure endowment has a single payment at time $n$, all dict values are **scalars**:

| Key | Description |
|-----|-------------|
| `payment_time` | $n$ — the maturity time |
| `joint_survival_probability` | ${}_{n}p_x$ (single-life) or $\prod_i {}_{n}p_{x_i}$ (joint) |
| `discount_factor` | $v^n$ |
| `present_value` | $v^n \cdot {}_{n}p_x$ |

:::{note}
Both `discrete_precision` and `discrete_simplified` return this same four-key structure.
For `continuous_precision` and `continuous_simplified`, see the sections below.
:::

---

(continuous-mode-keys)=
## Continuous mode keys

For `continuous_precision`, methods return **integration grid arrays** over $[d, d+n]$
instead of per-payment arrays:

**Annuity (`continuous_precision`)**:

All arrays are evaluated on a linearly-spaced integration grid over $[d,\, d+n]$.

| Key | Description |
|-----|-------------|
| `payment_time` | Grid points $t$ over $[d,\, d+n]$ (equals $[0, n]$ when $d = 0$) |
| `interest_rate` | Effective annual rates at each grid point |
| `discount_factor` | $v^t$ at each grid point |
| `survival_probability` | ${}_{t}p_x$ (raw $\ell_x$, no rounding applied) |
| `growth` | Growth factors at each grid point |
| `integrand` | $v^t \cdot {}_t p_x \cdot g(t)$ — the integrand of $\bar{a}_x$ |

:::{note}
The continuous annuity dict does **not** include a `"present_value"` key.
Reconstruct $\bar{a}_x$ by integrating the `integrand` array:

```python
import numpy as np
from lactuca import LifeTable, config

lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)
config.calculation_mode = "continuous_precision"
result = lt.äx(65, n=20, return_flows=True)
pv = np.trapezoid(result["integrand"], result["payment_time"])
print(round(pv, 4))  # 13.2532
config.calculation_mode = "discrete_precision"
```
:::

**Insurance (`continuous_precision`)**:

All arrays are evaluated on the same integration grid; `present_value` is a scalar.

| Key | Description |
|-----|-------------|
| `t_payment` | Integration grid points |
| `survival` | ${}_{t}p_x$ |
| `force_mortality` | $\mu_{x+t}$ (numerically derived) |
| `discount` | $v^t$ |
| `growth` | Benefit growth factors |
| `integrand` | $v^t \cdot {}_t p_x \cdot \mu_{x+t} \cdot g(t)$ |
| `present_value` | **Scalar** — trapezoidal integral $\bar{A}_x$ |

**Endowment (`continuous_precision`)**:

The grid spans $[0, n]$ (endowments have no deferral concept).
`payment_time`, `integral`, and `present_value` are scalars; all other values are arrays.

| Key | Description |
|-----|-------------|
| `payment_time` | $n$ — maturity time (scalar) |
| `time_grid` | Integration grid $t \in [0, n]$ |
| `joint_survival_probability` | ${}_t p_x$ (or joint-life equivalent) at each grid point |
| `combined_force_mortality` | Combined force of mortality $\sum_i \mu_i(t)$ at each grid point |
| `force_interest` | Force of interest $\delta(t)$ at each grid point |
| `combined_integrand` | $\sum_i \mu_i(t) + \delta(t)$ — integrand of $-\ln({}_{n}E_x)$ |
| `integral` | **Scalar** — $\int_0^n [\sum_i \mu_i(t) + \delta(t)]\,\mathrm{d}t$ (trapezoidal) |
| `present_value` | **Scalar** — $e^{-\text{integral}} = {}_{n}E_x$ |

---

(simplified-modes-nested-structure)=
## Simplified modes — nested structure

`discrete_simplified` and `continuous_simplified` return **nested dicts** for annuity
and insurance methods.
For **annuity** methods the inner `"due"` and `"immediate"` components are
themselves full `discrete_precision` flow dicts.
For **insurance** methods the structure differs — see below.
For **endowment** methods, both modes return flat scalar dicts — see below.

**`discrete_simplified`**:

```python
from lactuca import LifeTable, config

lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)
config.calculation_mode = "discrete_simplified"
result = lt.äx(65, m=12, return_flows=True)

due_flows    = result["due"]           # full discrete_precision dict for annual äx
imm_flows    = result["immediate"]     # full discrete_precision dict for annual ax
pv_due       = result["pv_due"]        # scalar: sum of due component
pv_immediate = result["pv_immediate"]  # scalar: sum of immediate component
coef_due     = result["coef_due"]      # Woolhouse weight for due component
coef_imm     = result["coef_immediate"]
pv_final     = result["interpolated"]  # scalar PV: coef_due*pv_due + coef_imm*pv_immediate

print(f"coef_due={coef_due:.4f}, coef_imm={coef_imm:.4f}")  # 0.5417, 0.4583
print(round(pv_final, 4))  # 15.6316

config.calculation_mode = "discrete_precision"  # restore
```

:::{note}
`"pv_due"` and `"pv_immediate"` are only present when `m > 1`.
For `m = 1` the method delegates to `discrete_precision` and the returned dict has only
five keys: `"due"`, `"immediate"`, `"coef_due"` (= 1.0), `"coef_immediate"` (= 0.0),
and `"interpolated"`. In this case `"due"` and `"immediate"` alias the **same** underlying
dict — mutations to one will be reflected in the other.
:::

**`continuous_simplified`**:

```python
from lactuca import LifeTable, config

lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)
config.calculation_mode = "continuous_simplified"

# Integer n — fractional sub-dict is empty
result = lt.äx(65, n=20, return_flows=True)
print(list(result.keys()))
# ['due', 'immediate', 'interpolated', 'fractional']
print(round(result["interpolated"], 4))  # 13.2555
print(result["fractional"])              # {}

# Fractional n — fractional sub-dict holds the two-point quadrature tail
result2 = lt.äx(65, n=20.5, return_flows=True)
frac = result2["fractional"]
print(list(frac.keys()))
# ['payment_time', 'interest_rate', 'discount_factor',
#  'survival_probability', 'growth', 'present_value']
print(round(result2["interpolated"], 4))  # 13.4242

config.calculation_mode = "discrete_precision"  # restore
```

| Key | Type | Description |
|-----|------|-------------|
| `"due"` | dict | Annual annuity-due flows over the **integer portion** $k = \lfloor n \rfloor$ (a full `discrete_precision` dict) |
| `"immediate"` | dict | Annual annuity-immediate flows over $k = \lfloor n \rfloor$ (a full `discrete_precision` dict) |
| `"interpolated"` | float | $\bar{a}_{x:\overline{n}\|} \approx \tfrac{1}{2}\bigl(\ddot{a}_{x:\overline{k}\|} + a_{x:\overline{k}\|}\bigr) + \tfrac{s}{2}\bigl(v^k\,{}_kp_x + v^n\,{}_np_x\bigr)$ where $k = \lfloor n \rfloor$, $s = n - k$; reduces to $\tfrac{1}{2}\bigl(\ddot{a}_{x:\overline{n}\|} + a_{x:\overline{n}\|}\bigr)$ when $n \in \mathbb{Z}$ — see {doc}`calculation_modes` for derivation |
| `"fractional"` | dict | Two-point trapezoidal flows for the fractional tail $[k,\,n]$; empty dict `{}` when $n$ is an integer |

:::{note}
When `"fractional"` is populated (fractional $n$), its six inner arrays are **Python lists of two
elements** — not NumPy arrays.  Use `sum(frac["present_value"])` or
`np.add(*frac["present_value"])` rather than `frac["present_value"].sum()`.
:::

**Insurance (`discrete_simplified`, `m > 1`)**:

For insurance methods the Woolhouse approximation calculates $A_x^{(m)}$ by interpolating
between two annual values at ages $x$ and $x+1$.  The structure is flat — no `"due"`/`"immediate"`:

| Key | Type | Description |
|-----|------|-------------|
| `"Ax1"` | dict | Full `discrete_precision` dict computed at age $x$ (annual, m=1) |
| `"Ax2"` | dict | Full `discrete_precision` dict computed at age $x+1$ (annual, m=1) |
| `"factor"` | float | Woolhouse weight $\frac{m-1}{2m}$ |
| `"present_value"` | float | Scalar $A_x^{(m)} \approx A_{x}^{(1)} + \tfrac{m-1}{2m}\bigl(A_{x+1}^{(1)} - A_{x}^{(1)}\bigr)$ |

**Insurance (`continuous_simplified`)**:

Interpolates $\bar{A}_x$ as the arithmetic mean of two `continuous_precision` evaluations —
one at age $x$, one at age $x+1$.  All three dict values are **scalars**:

| Key | Type | Description |
|-----|------|-------------|
| `"pv_x"` | float | $\bar{A}_x$ from `continuous_precision` |
| `"pv_x1"` | float | $\bar{A}_{x+1}$ from `continuous_precision` |
| `"present_value"` | float | $\tfrac{1}{2}(\bar{A}_x + \bar{A}_{x+1})$ |

**Endowment (`discrete_simplified`)**:

Returns the same four-key flat scalar structure as `discrete_precision` — see
[Pure-endowment cash-flow keys](#pure-endowment-cash-flow-keys).

**Endowment (`continuous_simplified`)**:

Uses average-force approximation ${}_{n}E_x \approx e^{-n(\bar{\delta} + \sum_i \bar{\mu}_i)}$.
All five dict values are **scalars**:

| Key | Type | Description |
|-----|------|-------------|
| `"payment_time"` | float | Maturity time $n$ |
| `"average_interest_force"` | float | $\bar{\delta} = -\ln(v^n)/n$ |
| `"average_mortality_force"` | float | $\sum_i \bar{\mu}_i = \sum_i(-\ln({}_np_{x_i})/n)$ |
| `"total_average_force"` | float | $\bar{\delta} + \sum_i \bar{\mu}_i$ |
| `"present_value"` | float | $\exp\!\bigl(-n\cdot(\bar{\delta}+\sum_i\bar{\mu}_i)\bigr)$ |

---

## Decomposing and verifying a present value

`return_flows=True` makes it possible to inspect every factor that contributed to a
present value and verify that they compose exactly as expected.  This kind of
component-level traceability is the building block for model validation tasks —
such as auditing an actuarial cash-flow model or cross-checking a single benefit
stream inside a larger IFRS 17 / Solvency II projection.

:::{note}
A real BEL under IFRS 17 / Solvency II covers a full portfolio (many policyholders),
multiple cash-flow types (benefits, premiums, expenses), multiple decrement causes
(mortality, lapse, disability …), and a Risk Adjustment / Risk Margin on top of the
pure present value.  The example below isolates **one benefit stream for a single
policy**: it demonstrates the decomposition pattern, not a complete BEL calculation.
:::

```python
import numpy as np
from lactuca import LifeTable

lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)

x, n, m = 65, 20, 12
flows = lt.äx(x, n=n, m=m, return_flows=True)

# 240 entries — one per payment (m=12 per year × n=20 years)
t           = flows["payment_time"]          # t_j = j/12, j = 0, …, 239
v_t         = flows["discount_factor"]       # v^{t_j}
tpx         = flows["survival_probability"]  # _{t_j}p_x
g_t         = flows["growth"]                # 1.0 everywhere (no gr=)
adj         = flows["payment_adjustment"]    # 1.0 for all full payments
pv_per_pmnt = flows["present_value"]         # = tpx * v_t * g_t * adj / m

pv = pv_per_pmnt.sum()
print(f"PV: {pv:.4f}")  # PV: 13.2805

# Verify that each element matches its formula component-by-component
reconstructed = tpx * v_t * g_t * adj / m
np.testing.assert_allclose(pv_per_pmnt, reconstructed, rtol=1e-12)
print("Components verified ✓")
```

---

## `summary()` — audit snapshots

Every main Lactuca object implements `summary()`, returning a human-readable snapshot
of its current state.  This is the standard audit header for IFRS 17 and Solvency II
model documentation.

| Class | What `summary()` shows |
|-------|------------------------|
| `LifeTable` | Table name, sex, ω, active modifications, decimals, sample $q_x$ values |
| `DecrementTable` | Same header as `LifeTable` plus active decrement columns |
| `DisabilityTable` | Table name, age range, sample incidence/recovery rates |
| `ExitTable` | Table name and lapse-rate sample |
| `InterestRate` | Rate type (constant / piecewise), rate values, scenario name if multi-scenario |
| `GrowthRate` | Growth type (geometric / arithmetic), rate values |
| `TableBuilder` | Builder configuration and included tables |

```python
from lactuca import LifeTable, InterestRate

lt = LifeTable("PASEM2020_Rel_1o", "m")
print(lt.summary())
# LifeTable: PASEM2020_Rel_1er.orden
# Sex: m
#   Table: PASEM2020_Rel_1er.orden  |  Type: life
#   Age range: 0-109  |  Valid sexes: f, m
#   Generational: False
# w (current): 109
# Modified: False
# Decimals: qx=15
# Sample qx values (first 5):
#   qx(0) = 0.0020038
#   ...
# Sample qx values (last 5):
#   ...
#   qx(109) = 1.0

ir = InterestRate(0.03)
print(ir.summary())
# InterestRate: constant rate = 0.030000
```

---

## See also

- {doc}`calculation_modes` — which mode to choose for regulatory calculations
- {doc}`prospective_reserve` — the `ts=` parameter and mid-year reserve calculations
- {doc}`../formulas` — mathematical definitions of each quantity
- {doc}`numerical_precision` — float64 precision policy for the arrays returned

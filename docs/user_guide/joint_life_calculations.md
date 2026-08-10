# Joint-Life Calculations

Lactuca supports actuarial calculations on **two or more simultaneously alive lives** — the
foundation for pricing and reserving joint annuities, reversionary annuities, and life-insurance
products with joint-life or first-death triggers.

All joint-life methods are available both as `LifeTable` instance methods and as standalone
functions importable directly from `lactuca`.

For the full mathematical definitions of joint-life symbols see {doc}`../formulas`.

---

## Independence assumption

All joint-life calculations in Lactuca rest on the **mutual independence** of the future
lifetimes.  For lives $(x)$, $(y)$, $(z)$, … the joint survival probability factorises as:

$${}_{t}p_{xy\ldots} = {}_{t}p_x \cdot {}_{t}p_y \cdot {}_{t}p_z \cdots$$

This means the death of one life does **not** alter the survival probabilities of the
remaining lives.  Independence is the standard assumption in actuarial pricing of joint
products.

:::{note}
If the portfolio exhibits **common-shock mortality** (e.g. spouses with correlated
lifetimes), the factorised formula underestimates the joint survival probability.  Such
cases require specialist models beyond the scope of this library.
:::

### Numerical evaluation (log domain)

Joint survival is computed as ${}_t p_{xy\ldots} = \exp\bigl(\sum_i \ln {}_t p_{x_i}\bigr)$
rather than multiplying probabilities directly.  This is actuarially equivalent under
independence (including $t \le 1$ and $t = 0$).  See {doc}`numerical_precision`.

---

## Status types

Lactuca implements two primary joint statuses directly:

**Joint life** (`äxy`, `axy`, `äjoint`, etc.)
: Payments continue while **all** named lives survive simultaneously.  Under independence:
  ${}_{t}p_{xy} = {}_{t}p_x \cdot {}_{t}p_y$.

**First death** (`Axy`, `Axyz`, `Afirst`)
: The insurance benefit is payable on the **first death** among the named lives.

**Last survivor** — no dedicated method.  All last-survivor quantities are derivable
from joint-life and single-life results via classical inclusion-exclusion identities.
See {ref}`derivable_formulas` for ready-to-use formulas.

---

## Joint annuities

`äxy` (annuity-due) and `axy` (annuity-immediate) cover two lives.  For three lives
use `äxyz` / `axyz` (additional tables via `tables_yz`).  `äjoint` / `ajoint` accept
any number of lives (`tables_others` must hold exactly `len(ages) - 1` tables).

:::{tip}
Pass `interest_rate=` directly in the constructor when creating multiple instances at once,
instead of setting it individually on each object afterwards:

```python
# All four instances get interest_rate=0.03 in a single call
lt_m, lt_f, lt_m2, lt_f2 = LifeTable(
    "PER2020_Ind_1o",
    ("m", "f", "m", "f"),
    cohort=[1961, 1964, 1968, 1974],
    interest_rate=0.03,
)
```

See {ref}`vectorial-interest-rate` for the full broadcast and per-instance sequence syntax.
:::

```python
from lactuca import LifeTable

# Cohorts for valuation ≈ 2026: 65/62 → 1961/1964; extra lives ≈ 58/52
lt_m, lt_f, lt_m2, lt_f2 = LifeTable(
    "PER2020_Ind_1o",
    ("m", "f", "m", "f"),
    cohort=[1961, 1964, 1968, 1974],
    interest_rate=0.03,
)

# Two lives: äxy(65, 62) — annuity-due while both survive
a2 = lt_m.äxy([65, 62], table_y=lt_f)

# When table_y is omitted, the calling table is used for both lives
a2_same = lt_m.äxy([65, 62])

# Two lives, temporary (n=20) and monthly (m=12)
a2_temp = lt_m.äxy([65, 62], table_y=lt_f, n=20, m=12)

# Two lives, 10-year deferral
a2_def = lt_m.äxy([65, 62], table_y=lt_f, d=10)

# Two lives, annuity-immediate (payment at end of each period)
a2_imm = lt_m.axy([65, 62], table_y=lt_f)

# Three lives: äxyz(60, 55, 58)
a3 = lt_m.äxyz([60, 55, 58], tables_yz=[lt_f, lt_m2])

# Four lives
a4 = lt_m.äjoint([60, 55, 58, 52], tables_others=[lt_f, lt_m2, lt_f2])
```

---

## Joint pure endowment

`nExy` (two lives), `nExyz` (three lives), and `nEjoint` (any number of lives)
return the present value of 1 payable if *all* specified lives survive $n$ years:

$${}_{n}E_{xy} = v^n \cdot {}_{n}p_{xy}$$

Note: `n` is a **required** keyword argument for all joint pure endowment methods.

```python
from lactuca import LifeTable

lt_m, lt_f, lt_m2, lt_f2 = LifeTable(
    "PASEM2020_Rel_1o", ("m", "f", "m", "f"), interest_rate=0.03
)

# Two lives: both survive 10 years from ages 60 and 58
nExy_val = lt_m.nExy([60, 58], table_y=lt_f, n=10)

# Three lives: all three survive 10 years
nExyz_val = lt_m.nExyz([60, 58, 55], tables_yz=[lt_f, lt_m2], n=10)

# Four lives
nEjoint_val = lt_m.nEjoint([60, 58, 55, 52], tables_others=[lt_f, lt_m2, lt_f2], n=10)
```

---

## First-death insurance

`Axy` (two lives), `Axyz` (three lives), and `Afirst` (any number of lives) compute the
present value of a 1-unit benefit on the **first death**.  The timing within the year of
death is governed by `config.mortality_placement` (`"beginning"`, `"mid"` (default), or
`"end"`).

```python
from lactuca import LifeTable

lt_m, lt_f, lt_m2, lt_f2 = LifeTable(
    "PASEM2020_Rel_1o", ("m", "f", "m", "f"), interest_rate=0.03
)

# Two lives: Axy(60, 58) — whole-life first-death insurance
Axy_val = lt_m.Axy([60, 58], table_y=lt_f)

# Two lives, 20-year term
Axy_term = lt_m.Axy([60, 58], table_y=lt_f, n=20)

# Three lives: Axyz
Axyz_val = lt_m.Axyz([60, 58, 55], tables_yz=[lt_f, lt_m2])

# Four lives
Afirst_val = lt_m.Afirst([60, 58, 55, 52], tables_others=[lt_f, lt_m2, lt_f2])
```

---

## Functional API

All joint-life methods are re-exported from `lactuca` and can be imported directly.
The first argument is a sequence of tables; `ages` is a keyword argument:

```python
from lactuca import LifeTable, äxy, axy, Axy, nExy, äjoint, Afirst

lt_m, lt_f = LifeTable(
    "PER2020_Ind_1o", ["m", "f"], cohort=[1961, 1964], interest_rate=0.03
)
lt_m_risk, lt_f_risk = LifeTable(
    "PASEM2020_Rel_1o", ["m", "f"], interest_rate=0.03
)

# Functional form: pass a sequence of tables and keyword ages
a_func = äxy([lt_m, lt_f], ages=[65, 62])           # longevity / PER
A_func = Axy([lt_m_risk, lt_f_risk], ages=[65, 62])  # life-risk / PASEM
```

---

(derivable_formulas)=
## Derivable formulas

Lactuca provides **joint-life** (all survive) and **first-death** methods directly.
All other standard joint products follow from classical inclusion-exclusion identities.
No extra code is needed — combine the existing methods as shown below.

### Last-survivor annuity

A last-survivor annuity pays while **at least one** life survives.  The **inclusion-exclusion
principle** applied to the joint survival indicator
$\mathbf{1}(\text{at least one alive}) = \mathbf{1}(x\text{ alive}) + \mathbf{1}(y\text{ alive}) - \mathbf{1}(\text{both alive})$
yields, on taking actuarial present values:

For two lives:

$$\ddot{a}_{\overline{xy}} = \ddot{a}_x + \ddot{a}_y - \ddot{a}_{xy}$$

For three lives (inclusion-exclusion):

$$\ddot{a}_{\overline{xyz}} = \ddot{a}_x + \ddot{a}_y + \ddot{a}_z
  - \ddot{a}_{xy} - \ddot{a}_{xz} - \ddot{a}_{yz} + \ddot{a}_{xyz}$$

```python
from lactuca import LifeTable

lt_m, lt_f = LifeTable(
    "PER2020_Ind_1o", ["m", "f"], cohort=[1966, 1964], interest_rate=0.03
)

ax  = lt_m.äx(60)
ay  = lt_f.äx(62)
axy = lt_m.äxy([60, 62], table_y=lt_f)

# Last-survivor annuity: pays while at least one of (x) or (y) survives
a_last = ax + ay - axy
```

### Reversionary annuity

A reversionary annuity pays while $(y)$ survives **after** $(x)$ has already died:

$$\ddot{a}_{x|y} = \ddot{a}_y - \ddot{a}_{xy}$$

```python
from lactuca import LifeTable

lt_m, lt_f = LifeTable(
    "PER2020_Ind_1o", ["m", "f"], cohort=[1966, 1964], interest_rate=0.03
)

ay  = lt_f.äx(62)
axy = lt_m.äxy([60, 62], table_y=lt_f)

# Reversionary annuity: pays to (y=62) only after (x=60) has died
a_rev = ay - axy
```

### Last-death (second-death) insurance

A last-death insurance pays on the **last death** among the named lives.  For two lives:

$$A_{\overline{xy}} = A_x + A_y - A_{xy}$$

```python
from lactuca import LifeTable

lt_m, lt_f = LifeTable("PASEM2020_Rel_1o", ["m", "f"], interest_rate=0.03)

Ax_val  = lt_m.Ax(60)
Ay_val  = lt_f.Ax(62)
Axy_val = lt_m.Axy([60, 62], table_y=lt_f)

# Last-death insurance: benefit paid on the second (last) death
A_last = Ax_val + Ay_val - Axy_val
```

---

## Parameter reference

Joint-life annuity and first-death insurance methods (`äxy`, `axy`, `äjoint`,
`ajoint`, `äxyz`, `axyz`, `Axy`, `Axyz`, `Afirst`) share these keyword-only
parameters:

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `n` | `float \| None` | `None` | Maximum duration in years (permanent if `None`) |
| `d` | `float` | `0.0` | Deferral period in years |
| `m` | `int` | `1` | Payment / policy frequency per year |
| `ts` | `float` | `0.0` | Time shift for off-anniversary reserves |
| `ir` | `float \| InterestRate` | table default | Interest rate |
| `gr` | `float \| GrowthRate \| None` | `None` | Benefit growth rate |
| `return_flows` | `bool` | `False` | Scalar: engine dict (`discrete_precision` / `continuous_precision`: per-payment arrays; simplified modes: nested structure — see {doc}`inspecting_cashflows`); batch: portfolio flow dict (precision modes only) |

Pure endowment methods (`nExy`, `nExyz`, `nEjoint`) accept only `n` (required), `ts`,
`ir`, and `return_flows`. They do not accept `d`, `m`, or `gr`.

---

## Batch joint-life calculations

All joint-life methods support batch mode: pass arrays for **every** life's ages
simultaneously.  Results are `NDArray[float64]` of the same length.

With `calculation_mode='discrete_precision'`, you may pass a shared `cashflow_times` grid
to immediate annuities (`axy`, `axyz`, `ajoint`) and first-death insurance (`Axy`, `Axyz`,
`Afirst`) in batch — the schedule is vectorised across all policies.  Due joint annuities
(`äxy`, `äxyz`, `äjoint`) also accept a shared `cashflow_amounts` vector on the standard
due payment grid in batch (no `cashflow_times`).  See
{doc}`irregular_cashflows` and {doc}`batch_calculations` for constraints (shared grid,
grouping by product when schedules differ).

### Two-life batch

```python
from lactuca import LifeTable, äxy, Axy

# Longevity tables for joint annuities (representative cohorts for the age grid)
lt_m = LifeTable("PER2020_Ind_1o", "m", cohort=1966, interest_rate=0.03)
lt_f = LifeTable("PER2020_Ind_1o", "f", cohort=1971, interest_rate=0.03)
# Life-risk tables for first-death insurance
lt_m_risk = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)
lt_f_risk = LifeTable("PASEM2020_Rel_1o", "f", interest_rate=0.03)

x_ages = [60, 62, 65]
y_ages = [55, 58, 61]

# Joint annuity-due: payments while both lives survive
result_ann = äxy([lt_m, lt_f], ages=(x_ages, y_ages), n=20)

# First-death insurance (same mortality base for A and ä if computing net premium)
result_ins = Axy([lt_m_risk, lt_f_risk], ages=(x_ages, y_ages), n=20)
result_ann_risk = äxy([lt_m_risk, lt_f_risk], ages=(x_ages, y_ages), n=20)
net_premium = result_ins / result_ann_risk

# OOP equivalent — shared table for life x, table_y for life y
result_ann_oop = lt_m.äxy((x_ages, y_ages), table_y=lt_f, n=20)
result_ins_oop = lt_m_risk.Axy((x_ages, y_ages), table_y=lt_f_risk, n=20)
```

### Three-life batch

```python
from lactuca import äxyz

lt_z = LifeTable("PER2020_Ind_1o", "f", cohort=1976, interest_rate=0.03)

z_ages = [50, 55, 58]

result = äxyz(
    [lt_m, lt_f, lt_z],
    ages=(x_ages, y_ages, z_ages),
    n=20,
)
```

### *n*-life batch — joint annuities / pure endowments / first-death insurance

(`äjoint`, `ajoint`, `nEjoint` require all lives to survive; `Afirst` pays on first death.)

For the n-life methods, pass **all tables** as a single list and **all ages** as a list
of arrays — one per life:

```python
from lactuca import äjoint

result = äjoint(
    [lt_m, lt_f, lt_z],
    ages=[x_ages, y_ages, z_ages],
    n=20,
)  # NDArray shape (3,)
```

For a **per-policy multi-table dispatch** (each policy has its own set of tables),
use the **functional API** (`äjoint`, `ajoint`, `Afirst`, `nEjoint`). Pass one
inner list per **life** (not per policy); each inner list has length `N` (one
`LifeTable` per policy for that life):

```python
# Functional API — 3 policies, 3 lives; tables[j][i] = table for life j, policy i
tables = [
    [lt_m, lt_f, lt_m],   # life 0
    [lt_f, lt_m, lt_f],   # life 1
    [lt_z, lt_z, lt_m],   # life 2
]
result = äjoint(tables, ages=[x_ages, y_ages, z_ages], n=20)  # NDArray shape (3,)
```

On the **OOP** path (`lt.äjoint(ages, tables_others=…)`), pass a length-`N` list of
`(n_lives - 1)`-element table lists — one inner list per **policy** (see
{doc}`batch_calculations` § Multi-table batch).

See {doc}`batch_calculations` for full batch documentation, including aggregate
portfolio flows for BEL calculations.

---

## Inspecting flows (`return_flows`)

With `return_flows=True`, joint-life methods return the same dict structures as their
single-life counterparts — per-payment or integration grids for annuities and insurances;
pure endowments (`nExy`, `nExyz`, `nEjoint`) return a maturity-focused dict
(`joint_survival_probability`, `expected_cf`, …) instead of a payment schedule:

| Call mode | `return_flows=True` returns |
|-----------|----------------------------|
| **Scalar** — `ages[0]` is a single `float` | Per-payment engine dict (`time_grid`, `present_value`, …). See {doc}`inspecting_cashflows`. |
| **Batch** — every `ages[i]` is an array | Compact portfolio dict: `time_grid`, `expected_cf`, `pv_cf`, `total_pv`. See {doc}`batch_calculations`. |

Batch aggregate flows require `calculation_mode='discrete_precision'` or
`'continuous_precision'`. `discrete_simplified` and `continuous_simplified` raise
`ValueError` in batch mode when `return_flows=True`.

Per-policy benefit weighting in batch uses `benefits=` (shape `(N,)`), not
`cashflow_amounts`. The `cashflow_amounts` schedule is **shared** across all policies
in one call — for heterogeneous sums insured, scale via `benefits=` or post-multiply
unit flows. See {doc}`irregular_cashflows`.

---

## See also

- {doc}`../formulas` — full mathematical definitions, joint-life status probabilities
- {doc}`deferment` — deferment parameter `d`: formulas, code examples, deferred insurance, and fractional `d`
- {doc}`prospective_reserve` — interaction between `d` and the elapsed-time parameter `ts`
- {doc}`functional_api` — complete functional API reference
- {doc}`calculation_modes` — discrete vs. continuous and frequency effects on joint calculations
- {doc}`notation_glossary` — joint-life symbols $\ddot{a}_{xy}$, $\ddot{a}_{\overline{xy}}$
- {doc}`batch_calculations` — batch mode guide

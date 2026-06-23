# Batch Calculations

Lactuca supports **batch mode**: passing a list or array of ages — and optionally
per-policy values for `n`, `ir`, `m`, `d`, `ts`, `gr`, or `benefits` — to any calculation method
instead of a single scalar.  The same implementation path is used whether you price one
policy or one million — no iteration in Python is required.

:::{note}
**"Per-policy" is a generic term.**
Throughout this page, *policy* refers to any individual calculation unit:
an insurance policy, a pension plan member, a bond position, a loan, a
guaranteed annuity contract, or any other item in a portfolio.
The batch API makes no assumption about the nature of the underlying product.
:::

---

## Batch methods covered

All life-table and financial-annuity methods in Lactuca support batch mode:

| Method family | OOP | Functional |
|---|---|---|
| Life annuity-immediate | `lt.ax(ages, …)` | `ax(lt, ages, …)` |
| Life annuity-due | `lt.äx(ages, …)` | `äx(lt, ages, …)` |
| Life insurance (term / whole) | `lt.Ax(ages, …)` | `Ax(lt, ages, …)` |
| Pure endowment | `lt.nEx(ages, …)` | `nEx(lt, ages, …)` |
| Two-life joint | `lt.axy(…)`, `lt.äxy(…)`, `lt.Axy(…)`, `lt.nExy(…)` | `axy(…)`, `äxy(…)`, `Axy(…)`, `nExy(…)` |
| Three-life joint | `lt.axyz(…)`, `lt.äxyz(…)`, `lt.Axyz(…)`, `lt.nExyz(…)` | `axyz(…)`, `äxyz(…)`, `Axyz(…)`, `nExyz(…)` |
| *n*-life joint | `lt.ajoint(…)`, `lt.äjoint(…)`, `lt.Afirst(…)`, `lt.nEjoint(…)` | `ajoint(…)`, `äjoint(…)`, `Afirst(…)`, `nEjoint(…)` |
| Financial annuity (no mortality) | `ir.a(…)`, `ir.ä(…)` | — |

---

## When to use batch mode

| Scenario | Recommended approach |
|---|---|
| Single policy | Scalar `x` → returns `float` |
| Portfolio on one table, shared parameters | List/array `x` → `lt.ax([55, 60, 65])` |
| Portfolio on one table, per-policy `n`, `ir`, `d`, `ts`, `m`, or `gr` | List/array `x` + per-policy parameter lists → `lt.ax(ages, n=[30, 20, 15], ir=[0.02, 0.03, 0.03])` |
| Per-policy payment frequency with shared age (e.g. pricing grid over `m`) | Scalar `x` + array `m` → `lt.ax(65, n=20, m=[1, 6, 12])` — `m` alone triggers batch |
| Portfolio split across multiple tables (sex, cohort…) | Functional API with `table=[lt_m, lt_f, …]` |
| Two-life joint portfolio | `axy([lt_m, lt_f], (x_arr, y_arr))` |
| Three-life joint portfolio | `axyz([lt_x, lt_y, lt_z], (x_arr, y_arr, z_arr))` |
| *n*-life joint portfolio (`äjoint`, `ajoint`, `Afirst`, `nEjoint`) | Tables list + ages list of arrays, one per life |
| Per-policy duration with shared ages (e.g. pricing grid over `n`) | Scalar ages + array `n` → `lt.axy([65, 60], n=[10, 20, 30])` — `n` alone triggers batch |
| Aggregate expected cash flows (APV, BEL, PVDBO) | `return_flows=True` with array `x` (precision modes) |
| Portfolio BEL/PVDBO with per-policy sum insured | `benefits=sums_insured` + array `x` (scaled PVs in all modes; aggregate flows only in precision modes) |
| Portfolio with invalid records (robust processing) | `on_error='nan'` → returns `BatchResult(values, errors)` |
| Guaranteed annuity certain (no mortality) — per-policy terms / ALM / IFRS 17 | `ir.a(n=n_arr)` or `ir.ä(n=n_arr)` — see [Pure financial annuities](#pure-financial-annuities) |
| Benefit-weighted bond/loan portfolio aggregate cashflows | `ir.a(n=n_arr, benefits=face_values, return_flows=True)` — see [Benefit-weighted flows](#benefit-weighted-portfolio-cash-flows) |

---

## Parameter compatibility reference

Quick reference for combining batch parameters across API modes.
This table is a summary — each combination is explained in detail in the sections below.
"All batch modes" covers single-life batch (OOP), joint/n-life batch (OOP), and the functional multi-table variants.

| Parameter / combination | Single-life scalar | All batch modes |
|---|---|---|
| `return_flows=True` + precision modes | ✅ (per-policy dict) | ✅ |
| `return_flows=True` + simplified modes | ✅ (own dict, not batch-aggregable) | ❌ `ValueError` |
| `on_error='nan'` | ❌ `ValueError` (scalar `x`/`ages`) | ❌ `ValueError` if `return_flows=True`; ✅ `BatchResult` otherwise |
| `t_output=` | ❌ `ValueError` (batch-only parameter) | ✅ with `return_flows=True` in precision modes; ❌ `ValueError` if `return_flows=False` |
| `record_ids=` | ❌ `ValueError` (batch-only parameter) | ✅ identifies policies in `BatchErrorReport` |
| `benefits=` | ❌ `ValueError` | ✅ for scaled PVs in all modes; aggregate flows only in precision modes |
| `benefits=` + `on_error='nan'` | ❌ `ValueError` (scalar `x`/`ages`) | ✅ `BatchResult`; `NaN × benefit = NaN` |
| `benefits=` + simplified modes | ❌ `ValueError` (scalar `x`/`ages`) | ✅ `return_flows=False`; ❌ `ValueError` if `return_flows=True` |
| Per-policy (heterogeneous) `m`, `return_flows=False` | — | ✅ |
| Per-policy (heterogeneous) `m`, `return_flows=True` | — | ❌ `ValueError` |
| Per-policy (heterogeneous) `ir` or `gr`, any `return_flows` | — | ✅ |

:::{note}
**Simplified modes** (`discrete_simplified`, `continuous_simplified`) support
`return_flows=True` in scalar mode (returning a per-policy diagnostic dict with a different
schema), but raise `ValueError` in batch mode when `return_flows=True` is combined with
an array `x` or `ages`.  The exception is `ir.a()` / `ir.ä()`: they support
`return_flows=True` with batch `n` in all four calculation modes.

:::

---

## Return-type rules

Batch mode is triggered if **any** of `x`, `n`, `d`, `ts`, `ir`, `gr`, or `m` is a
list, tuple, or `ndarray` of ndim ≥ 1 — not only `x`.  For joint-life methods, the
same rule applies to `ages` and the other parameters.

| Input form | Return type |
|---|---|
| All parameters are `int` / `float` scalars or 0-d `ndarray` | `float` |
| Any parameter is a `list` or `tuple` — even length 1 | `NDArray[float64]` |
| Any parameter is a `numpy.ndarray` of ndim ≥ 1 | `NDArray[float64]` |
| Any parameter is a Pandas or Polars `Series` (any length ≥ 1) | `NDArray[float64]` |

:::{important}
A **length-1 list, tuple, or `ndarray` of shape `(1,)`** returns an `NDArray` of shape
`(1,)`, **not** a `float`.  This ensures downstream array operations behave correctly
regardless of portfolio size.
:::

---

(error-handling-in-batch-mode)=
## Error handling in batch mode

By default all batch functions raise a `ValueError` immediately if any record in the
input array is invalid (e.g. negative age, `n ≤ 0`, interest rate ≤ −1).  This is
the `on_error='raise'` behaviour — the same as scalar calls.

Set `on_error='nan'` to mark invalid records with `numpy.nan`; the function returns a
{class}`~lactuca.BatchResult` — a two-field named tuple — instead of a plain array:

| Field | Type | Description |
|---|---|---|
| `values` | `NDArray[float64]` | Computed values; `nan` at invalid positions |
| `errors` | `BatchErrorReport` | Structured report of all validation failures |

The `BatchErrorReport` fields are documented in [{class}`~lactuca.BatchErrorReport` reference](#batcherrorreport-reference) below.

### Basic usage

The examples below use the single-table OOP API (`lt.ax(...)`, covered in detail in
[Single-table batch](#single-table-batch)); `on_error` works identically in all batch
modes — single-table, multi-table functional, joint-life, and financial annuities.

```python
from lactuca import LifeTable

# PER2020_Ind_1o: longevity table for survival annuities
lt = LifeTable("PER2020_Ind_1o", "m", cohort=1960, interest_rate=0.03)

# Third record has an invalid age (-5)
x_arr = [55.0, 65.0, -5.0, 70.0]

values, report = lt.ax(x_arr, n=20, on_error="nan")

print(values)
# [14.2416  13.664      nan  12.9894]

print(report)
# BatchErrorReport: 1/4 invalid records

if report:
    print(f"{report.n_errors} of {report.n_total} records failed validation")
    print("Invalid positions:", report.invalid_indices)
    print("Messages:")
    for msg in report.messages:
        print("  -", msg.replace("\n", "\n    "))
```

### Tracking records with `record_ids`

Pass a sequence of identifiers via `record_ids` to label invalid records in the error
report:

```python
from lactuca import LifeTable

lt    = LifeTable("PER2020_Ind_1o", "m", cohort=1960, interest_rate=0.03)
x_arr = [55.0, 65.0, -5.0, 70.0]

policy_ids = ["P-001", "P-002", "P-003", "P-004"]

values, report = lt.ax(x_arr, n=20, on_error="nan", record_ids=policy_ids)

print(report.record_ids)      # ["P-003"]
print(report.invalid_indices) # [2]
```

Passing a DataFrame column as `record_ids` works the same way — no `.to_numpy()` needed:

```python
import pandas as pd
from lactuca import LifeTable

lt = LifeTable("PER2020_Ind_1o", "m", cohort=1960, interest_rate=0.03)

df = pd.DataFrame({
    "policy_id": ["P-001", "P-002", "P-003", "P-004"],
    "age":       [55.0, 65.0, -5.0, 70.0],
})

values, report = lt.ax(df["age"], n=20, on_error="nan", record_ids=df["policy_id"])

print(report.record_ids)      # ["P-003"]
print(report.invalid_indices) # [2]
```

:::{note}
`record_ids` accepts any sequence — plain list, Pandas Series, or Polars Series.
Values are used as policy identifiers in error reports and are not converted numerically.
:::

(batcherrorreport-reference)=
### `BatchErrorReport` reference

| Attribute | Type | Description |
|---|---|---|
| `n_errors` | `int` | Number of invalid records |
| `n_total` | `int` | Total records in the batch |
| `valid_mask` | `NDArray[bool]` | Boolean mask — `True` at valid positions |
| `invalid_indices` | `NDArray[int64]` | Positions of invalid records |
| `record_ids` | `list \| None` | User IDs for invalid records; `None` if not supplied |
| `messages` | `list[str]` | One error message per invalid record |
| `bool(report)` | `bool` | `True` when `n_errors > 0` |

`.to_dataframe()` returns a **Polars** `DataFrame` with columns `idx` and `record_id`:

```python
from lactuca import LifeTable

lt         = LifeTable("PER2020_Ind_1o", "m", cohort=1960, interest_rate=0.03)
x_arr      = [55.0, 65.0, -5.0, 70.0]
policy_ids = ["P-001", "P-002", "P-003", "P-004"]

_, report = lt.ax(x_arr, n=20, on_error="nan", record_ids=policy_ids)

df_errors = report.to_dataframe()
print(df_errors)
# shape: (1, 2) — columns: idx (int), record_id (any)
# ┌─────┬───────────┐
# │ idx ┆ record_id │
# │ i64 ┆ str       │
# ╞═════╪═══════════╡
# │ 2   ┆ P-003     │
# └─────┴───────────┘
```

### Filtering valid results

Use `valid_mask` to align results back to the original array without iterating:

```python
import numpy as np
from lactuca import LifeTable

lt    = LifeTable("PER2020_Ind_1o", "m", cohort=1960, interest_rate=0.03)
x_arr = np.array([55.0, 65.0, -5.0, 70.0])

values, report = lt.ax(x_arr, n=20, on_error="nan")

valid_values = values[report.valid_mask]
valid_ages   = x_arr[report.valid_mask]
print(valid_ages)
# [55. 65. 70.]
print(valid_values)
# [14.24158514 13.66404351 12.98942218]  (values at valid positions; nan position dropped)
```

### Import note

{class}`~lactuca.BatchResult` and {class}`~lactuca.BatchErrorReport` can be imported by name — useful for type
annotations or `isinstance` checks.  Tuple unpacking is the usual pattern for
everyday use:

```python
# Usual pattern — tuple unpacking, no explicit import needed
from lactuca import LifeTable

lt    = LifeTable("PER2020_Ind_1o", "m", cohort=1960, interest_rate=0.03)
x_arr = [55.0, 65.0, -5.0, 70.0]

values, report = lt.ax(x_arr, n=20, on_error="nan")
print(values)
# [14.24158514 13.66404351         nan 12.98942218]
print(report)
# BatchErrorReport: 1/4 invalid records
```

```python
# Explicit import — useful for type annotations or isinstance checks
from lactuca import LifeTable, BatchResult, BatchErrorReport

lt    = LifeTable("PER2020_Ind_1o", "m", cohort=1960, interest_rate=0.03)
x_arr = [55.0, 65.0, -5.0, 70.0]

result: BatchResult = lt.ax(x_arr, n=20, on_error="nan")
values = result.values                    # NDArray[float64]
report: BatchErrorReport = result.errors  # structured validation report
print(values)
# [14.24158514 13.66404351         nan 12.98942218]
print(report)
# BatchErrorReport: 1/4 invalid records
```

:::{warning}
`on_error='nan'` is **incompatible** with `return_flows=True`.  Combining both raises
a `ValueError` immediately regardless of the `on_error` setting.
:::

---

(single-table-batch)=
## Single-table batch

### Basic: array of ages

The simplest batch call passes only the ages; all other parameters are either left at
their defaults or shared by every policy.

```python
from lactuca import LifeTable, config

lt = LifeTable("PER2020_Ind_1o", "m", cohort=1960, interest_rate=0.03)
config.decimals.annuities = 4

ages = [55, 60, 65, 70]

# Whole-life annuity-immediate for four ages simultaneously
result = lt.ax(ages)
print(result)
# [20.4768 18.809  16.9732 14.9412]

config.reset_to_defaults()
```

### Per-policy parameters

All calculation parameters accept either a **scalar** (shared by all policies) or a
**list, tuple, or array of the same length as the age argument** (`x` or `ages`, depending
on the method; one value per policy) — **except
`cashflow_amounts` and `cashflow_times`**, which are always shared across all policies
in a single call and cannot be per-policy:

:::{rubric} Broadcasting rules
:::

The table below summarises how each input form is expanded to match the batch size N.
The same rule applies to every per-policy parameter (`n`, `ir`, `d`, `ts`, `m`, `gr`).

| Input form | Result |
|---|---|
| Scalar (`float`, `InterestRate`, `GrowthRate`, `int`, …) | Broadcast — same value used for all N policies |
| `list` or `tuple` of **length 1** | Broadcast — single element repeated N times |
| `list` or `tuple` of **length N** | One value per policy — index _i_ → policy _i_ |
| `numpy.ndarray` of shape `(1,)` | Broadcast |
| `numpy.ndarray` of shape `(N,)` | One value per policy |
| Pandas or Polars `Series` of **length 1** | Broadcast — no `.to_numpy()` needed |
| Pandas or Polars `Series` of **length N** | One value per policy — no `.to_numpy()` needed |
| Any other length | `ValueError` |

`list` and `tuple` are interchangeable for all per-policy parameters.
For `ir` and `gr`, list/tuple elements and object-dtype Pandas/Polars Series elements
are passed through without conversion, so `InterestRate` and `GrowthRate` objects
(including piecewise curves) are preserved intact.  A numeric `ir`/`gr` Series is
converted to per-policy floats.  Mixing object and numeric elements in one Series
raises `ValueError`.

| Parameter | Per-policy? | Notes |
|---|---|---|
| `x` or `ages` | ✅ Yes | The batch dimension — pass a list, array, or Pandas/Polars Series |
| `n` | ✅ Yes | Term in years; `None`, missing, or `np.inf` for whole-life; lists and Pandas/Polars `Series` |
| `ir` | ✅ Yes | Scalar `float`, `InterestRate`, NDArray, list/tuple, or Pandas/Polars Series (numeric float **or** object-dtype `InterestRate` instances, including piecewise curves) per policy |
| `m` | ✅ Yes | Payment frequency per year; accepts Pandas/Polars Series of ints |
| `d` | ✅ Yes | Deferral period in years; accepts Pandas/Polars Series |
| `ts` | ✅ Yes | Time shift in years; accepts Pandas/Polars Series |
| `gr` | ✅ Yes | Scalar `float`, `GrowthRate`, list/tuple, or Pandas/Polars Series (numeric float **or** object-dtype `GrowthRate` instances, including piecewise curves) per policy |
| `benefits` | ✅ Yes | Per-policy benefit weight; accepts Pandas/Polars Series (converted to `float64`). `return_flows=True` requires a precision mode |
| `cashflow_amounts` | ❌ Shared | Per-payment-period amount schedule — same for all policies |
| `cashflow_times` | ❌ Shared | Payment timing grid — same for all policies |
| `t_output` | ❌ Shared | Reporting time grid for `return_flows=True`; requires `return_flows=True` — raises `ValueError` otherwise |

:::{note}
**Whole-life duration (`n`).**  Scalar `n=None` means whole-life for every policy
(same as in scalar mode).  In per-policy vectors — `list`, `tuple`, or a Pandas/Polars
`Series` passed as `n` — a missing value at index *i* also denotes whole-life for
policy *i* (internally mapped to `np.inf`).  This includes:

- Python `None` in lists or in `dtype=object` Series (B2)
- `NaN` / null in numeric Series built from `[25, None, 20, …]` DataFrame columns (B3)

`np.inf` in a numeric array is equivalent.  Pure endowments (`nEx`, `nExy`, …) require
a finite positive term and reject whole-life sentinels.  Explicit `np.nan` inside a
bare `numpy.ndarray` (not a Series) is **not** remapped — it remains invalid.
:::

(multi-scenario-ir-gr-batch)=
:::{note}
**Multi-scenario `ir` and `gr`.**  When several policies share the same
`InterestRate` or `GrowthRate` container (scalar `ir=ir`, default `lt.interest_rate`,
or `gr=[gr, gr, gr]`), each batch call uses the **active scenario at call time**.
Building the list does not snapshot the scenario.  Use `ir.copy()` / `gr.copy()` per
policy when you need independent scenario state.  See {ref}`interest-rate-scenarios-lifetable`.
:::

```python
import numpy as np
from lactuca import GrowthRate, InterestRate, LifeTable

ir = InterestRate({"base": 0.02, "piecewise": ([5, 10], [0.01, 0.02, 0.025])})
lt = LifeTable("PASEM2020_Gen_2o", "m")
ages = np.array([50.0, 55.0, 60.0])

ir.active_scenario = "base"
batch_base = lt.ax(ages, n=20, ir=ir)

ir.active_scenario = "piecewise"
batch_piecewise = lt.ax(ages, n=20, ir=ir)   # differs from batch_base

gr = GrowthRate({"base": 0.02, "stress": 0.04})
gr.active_scenario = "base"
pv_base = lt.ax(ages, n=20, ir=0.03, gr=[gr, gr, gr])

gr.active_scenario = "stress"
pv_stress = lt.ax(ages, n=20, ir=0.03, gr=[gr, gr, gr])   # differs from pv_base
```

```python
from lactuca import LifeTable, InterestRate, config

lt   = LifeTable("PER2020_Ind_1o", "m", cohort=1960, interest_rate=0.03)
ages = [55, 60, 65, 70]
config.decimals.annuities  = 4
config.decimals.insurances = 4

# Per-policy term n (different residual durations — benefit annuity at each retirement)
result = lt.ax(ages, n=[30, 20, 15, 10], ir=0.03)
print(result)
# [18.1154 14.0231 11.2888  8.1347]

# Per-policy interest rate (e.g. different technical rates for benefit valuation)
result = lt.ax(ages, n=20, ir=[0.02, 0.03, 0.03, 0.04])
print(result)
# [15.6241 14.0231 13.664  11.9456]

# Per-policy deferral d (deferred pensions — each member retires in a different year)
result = lt.ax([45, 50, 55, 60], d=[20, 15, 10, 5], n=20, ir=0.03)
print(result)
# [ 7.1147  8.308   9.7507 11.5041]

# Per-policy time shift ts (policies valued at different offsets within the year)
result = lt.ax(ages, ts=[0.0, 0.25, 0.5, 0.75], n=20, ir=0.03)
print(result)
# [14.2416 13.8962 13.4158 12.6044]

# Per-policy growth rate gr (growing pension benefit, e.g. CPI-indexed)
result = lt.ax(ages, n=20, gr=[0.01, 0.02, 0.015, 0.0], ir=0.03)
print(result)
# [15.4985 16.6236 15.4759 12.9894]

# Per-policy sum insured via benefits=
# (uses PASEM2020_Rel_1o: mortality table for the insurance/death benefit component)
lt_risk = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)
sums     = [100_000, 150_000, 200_000, 80_000]

# Approach A: aggregate flows (precision modes only)
flows = lt_risk.Ax(ages, n=20, ir=0.03, return_flows=True, benefits=sums)
print(flows["total_pv"])
# 121372.71533714952

# Approach B: per-policy scaled PVs (all modes)
scaled = lt_risk.Ax(ages, n=20, ir=0.03, benefits=sums)
print(scaled)
# [11306.2096 25702.2292 52610.7319 31753.5446]
print(float(scaled.sum()))
# 121372.71530000001

# Manual equivalent: unit APV × sum insured
# Note: unit_apv is rounded to config.decimals.insurances = 4 decimal places,
# so the products are less precise than Approach B. In this example the sums
# are multiples of 1 000, so the rounded APVs produce exact integers.
unit_apv = lt_risk.Ax(ages, n=20, ir=0.03)   # rounded NDArray, shape (4,)
print(unit_apv)
# [0.1131 0.1713 0.2631 0.3969]
manual = unit_apv * sums
print(manual)
# [11310.  25695.  52620.  31752.]
print(float(manual.sum()))
# 121377.0

# Per-policy interest rate as list of InterestRate objects (e.g. piecewise curves per policy)
lt   = LifeTable("PER2020_Ind_1o", "m", cohort=1960, interest_rate=0.03)
ir_p1 = InterestRate(terms=[5.0, 10.0], rates=[0.015, 0.025, 0.035])  # piecewise curve
ir_p2 = InterestRate(0.03)                                              # flat rate
ir_p3 = InterestRate(terms=[10.0], rates=[0.02, 0.04])                 # two-segment curve
ir_p4 = InterestRate(0.04)                                              # flat rate

result = lt.ax(ages, n=20, ir=[ir_p1, ir_p2, ir_p3, ir_p4])
print(result)
# [15.4769 14.0231 14.3672 11.9456]

config.reset_to_defaults()
```

:::{tip}
Reusing the same {class}`lactuca.InterestRate` object for multiple policies with identical
rates is more efficient than creating separate instances.
:::

:::{note}
To price policies with **different payment schedules**, use the per-product grouping pattern
in the next section — `cashflow_times` is shared by all policies in a single call.

In `discrete_precision`, a **shared** irregular `cashflow_times` grid is vectorised in
batch for single-life (`ax`, `Ax`) and multi-life (`axy`, `axyz`, `ajoint`, `Axy`,
`Axyz`, `Afirst`) — see {doc}`irregular_cashflows` for performance notes.  Annuity-due
methods (`äx`, `äxy`, `äxyz`, `äjoint`) do not accept `cashflow_times`.
:::

### Custom payment schedules: per-product grouping

`cashflow_times` is shared by all policies in a single call — per-policy timing grids
are not supported.  The recommended pattern is to **group by product type** and make
one batch call per group:

```python
import numpy as np
from lactuca import LifeTable, ax, config, payment_times

lt = LifeTable("PER2020_Ind_1o", "m", cohort=1960, interest_rate=0.03)
config.decimals.annuities = 4

ct_monthly = payment_times(n=20, m=12)  # 240 monthly times: [1/12, 2/12, ..., 20.0]
PRODUCT_SCHEDULES = {
    "monthly_pension":  {"cashflow_times": ct_monthly,
                        "cashflow_amounts": np.ones(len(ct_monthly)) * 1_000.0},
    "annual_lump_sum": {"cashflow_times": [1.0, 2.0, 3.0],
                        "cashflow_amounts": [5_000.0, 5_000.0, 5_000.0]},
    "bullet":          {"cashflow_times": [10.0],
                        "cashflow_amounts": [50_000.0]},
}

ages        = np.array([55, 60, 62, 65, 68, 70])
product_ids = np.array(["monthly_pension", "annual_lump_sum", "monthly_pension",
                        "bullet", "annual_lump_sum", "monthly_pension"])

result = np.empty(len(ages), dtype=np.float64)

for prod, sched in PRODUCT_SCHEDULES.items():
    mask = product_ids == prod          # boolean mask
    result[mask] = ax(lt, ages[mask], **sched)

print(result)
# [173665.9228  14023.4219 169770.8149  34599.6658  13955.7438 159501.8063]
config.reset_to_defaults()
```

### Arithmetic on batch results

Batch results are plain NumPy arrays — standard arithmetic gives element-wise operations
with no additional API:

```python
from lactuca import LifeTable, config

lt   = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)
ages = [55, 60, 65, 70]
config.decimals.annuities  = 4
config.decimals.insurances = 4

Ax_       = lt.Ax(ages, n=20, ir=0.03)
a_due     = lt.äx(ages, n=20, ir=0.03)    # annuity-due: premiums paid at start of year
risk_premium = Ax_ / a_due                # net annual premium — Equivalence Principle: P = Ax / äx
print(Ax_)
# [0.1131 0.1713 0.2631 0.3969]
print(a_due)
# [14.5608 14.1829 13.5829 12.609 ]
print(risk_premium)
# [0.00776743 0.01207792 0.01936994 0.03147752]

config.reset_to_defaults()
```

---

(multi-table-batch-functional-api)=
## Multi-table batch (functional API)

:::{important}
Multi-table batch requires the **functional API** (`ax(tables, ages, …)`, `Ax(tables, ages, …)`, …).
The **OOP instance methods** (`lt.ax(ages, …)`) are bound to a single table and cannot
accept a list of tables — use them only for single-table batch.
:::

When policies belong to **different tables** — for example, a mixed-sex portfolio — pass a
list of `LifeTable` instances as the first argument to any functional-API function.  Each
element corresponds to one policy.

```python
from lactuca import LifeTable, ax, config

# Zip mode (cartesian=False, default): pairs sex[i] with cohort 1960 → 2 instances
lt_m, lt_f = LifeTable("PER2020_Ind_1o", ["m", "f"], cohort=1960, interest_rate=0.03)

ages      = [60, 62, 65, 68]
sexes     = ["m", "f", "m", "f"]
table_map = {"m": lt_m, "f": lt_f}
tables    = [table_map[sex] for sex in sexes]   # one table per policy

config.decimals.annuities = 4
result = ax(tables, ages, n=20)     # NDArray shape (4,)
print(result)
# [14.0231 14.3807 13.664  14.0408]
config.reset_to_defaults()
```

:::{note}
**`cohort` in multi-table examples**

The examples above use `cohort=1960` for all instances as a simplification.  In practice,
each insured has their own year of birth, so a portfolio with mixed cohorts typically
requires one `LifeTable` per unique `(sex, cohort)` combination.  See
[Cohort and duration](#cohort-and-duration-instance-level-properties) for the recommended patterns.
:::

### Building a lookup dict with `return_dict=True`

For **study grids** and **parameterised valuation runs**, pass `return_dict=True` to the
constructor to obtain a mapping from {class}`lactuca.TableKey` to `LifeTable` in a single call.
Combine with `cartesian=True` to build the full cross-product at once:

```python
from lactuca import LifeTable, TableKey, äx, config

# Build all (table, sex, cohort) combinations — 2 × 2 × 41 = 164 instances
grid = LifeTable(
    ["PER2020_Ind_1o", "PER2020_Col_2o"],
    ["m", "f"],
    cohort=range(1940, 1981),
    cartesian=True,
    return_dict=True,
    interest_rate=0.03,
)

# O(1) dict lookup by structured key
lt = grid[TableKey("PER2020_Ind_1o", "m", 1960)]

config.decimals.annuities = 4
print(lt.äx(65))
config.reset_to_defaults()
```

:::{tip}
`return_dict=True` also works in zip mode (`cartesian=False`, the default) for heterogeneous
sequences where key-based lookup is more readable than positional indexing:

```python
from lactuca import LifeTable, TableKey

tables = LifeTable(
    ["PER2020_Ind_1o", "PER2020_Col_2o"],
    ["m", "f"],
    cohort=[1960, 1965],
    return_dict=True,
)
lt = tables[TableKey("PER2020_Ind_1o", "m", 1960)]
```
:::

:::{warning}
**`cartesian=True` is for study grids, not portfolio processing.**
It generates every combination regardless of whether policies exist for it.
For a production portfolio, use the groupby pattern: create one `LifeTable` per unique
`(table_name, sex, cohort, duration)` group and call the batch API once per group.
:::

---

## Joint-life batch

For two-life joint calculations, pass arrays for **both** lives via `ages`:

```python
from lactuca import LifeTable, axy, config

# Zip mode (cartesian=False, default): pairs "m" and "f" with cohort 1960 → 2 instances
lt_m, lt_f = LifeTable("PER2020_Ind_1o", ["m", "f"], cohort=1960, interest_rate=0.03)

x_ages = [60, 62, 65]
y_ages = [55, 58, 61]

config.decimals.annuities = 4
# [lt_m, lt_f]: one table per life (life x uses lt_m, life y uses lt_f)
result = axy([lt_m, lt_f], (x_ages, y_ages), n=20)  # NDArray shape (3,)
print(result)
# [13.7359 13.5622 13.266 ]
config.reset_to_defaults()
```

For three lives, pass a 3-element list of tables to `axyz` / `Axyz` / `äxyz`:

```python
from lactuca import LifeTable, axyz, config

# Zip mode (cartesian=False, default): 3 instances paired positionally with cohort 1960
lt_x, lt_y, lt_z = LifeTable("PER2020_Ind_1o", ["m", "f", "m"], cohort=1960, interest_rate=0.03)
x, y, z = ([60, 65], [55, 60], [50, 55])

config.decimals.annuities = 4
result = axyz([lt_x, lt_y, lt_z], ages=(x, y, z), n=20)
print(result)
# [13.323  12.7634]
config.reset_to_defaults()
```

For *n*-life calculations (`äjoint`, `ajoint`, `Afirst`, `nEjoint`), pass all tables
as a single list and all ages as a list of arrays — one array per life:

```python
from lactuca import LifeTable, ajoint, config

# Zip mode (cartesian=False, default): 3 instances paired positionally with cohort 1960
lt_x, lt_y, lt_z = LifeTable("PER2020_Ind_1o", ["m", "f", "m"], cohort=1960, interest_rate=0.03)
x, y, z = ([60, 65], [55, 60], [50, 55])

config.decimals.annuities = 4
result = ajoint([lt_x, lt_y, lt_z], ages=[x, y, z], n=20)
print(result)
# [13.323  12.7634]
config.reset_to_defaults()
```

For **per-policy multi-table dispatch** (each policy has its own set of tables),
each element of the tables list must itself be a list of N `LifeTable` instances —
one per policy:

```python
from lactuca import LifeTable, ajoint, config

# Zip mode (cartesian=False, default): pairs "m" and "f" with cohort 1960 → 2 instances
lt_m, lt_f = LifeTable("PER2020_Ind_1o", ["m", "f"], cohort=1960, interest_rate=0.03)
table_map = {"m": lt_m, "f": lt_f}

x, y, z = ([60, 65, 62], [55, 60, 58], [50, 55, 52])

# In practice, per-policy sex arrays drive table selection:
sex_x = ["m", "f", "m"]   # sex of life 0, one entry per policy
sex_y = ["f", "m", "f"]   # sex of life 1, one entry per policy
sex_z = ["m", "m", "m"]   # sex of life 2, one entry per policy

# Build the tables structure with a dict lookup + nested comprehension
tables = [[table_map[s] for s in sex_arr] for sex_arr in [sex_x, sex_y, sex_z]]

config.decimals.annuities = 4
result = ajoint(tables, [x, y, z], n=20)  # NDArray shape (3,)
print(result)
# [13.323  12.9092 13.1015]
config.reset_to_defaults()
```

**Robust batch with invalid records (`on_error='nan'`):**

`on_error='nan'` and `record_ids` are supported in all batch modes — single-table OOP
and multi-table functional API alike.  Invalid records produce NaN entries; a
`BatchResult` is returned for the portfolio (see [Error handling](#error-handling-in-batch-mode) for the full API):

```python
from lactuca import LifeTable, axy, config

# Zip mode (cartesian=False, default): pairs "m" and "f" with cohort 1960 → 2 instances
lt_m, lt_f = LifeTable("PER2020_Ind_1o", ["m", "f"], cohort=1960, interest_rate=0.03)
table_map = {"m": lt_m, "f": lt_f}

# 4 couples; third couple has an invalid age (data quality issue)
x          = [60,   65,   -1,  62]
y          = [55,   60,   58,  57]
sex_x      = ["m", "f", "m", "m"]
sex_y      = ["f", "m", "f", "f"]
tables     = [[table_map[s] for s in sex] for sex in [sex_x, sex_y]]
policy_ids = ["P001", "P002", "P003", "P004"]

config.decimals.annuities = 4
result = axy(tables, (x, y), n=20, on_error="nan", record_ids=policy_ids)
values, report = result
print(values)            # [13.7359 13.4507     nan 13.5816]
print(report.n_errors)   # 1
print(report.record_ids) # ['P003']
config.reset_to_defaults()
```

### Per-policy mortality tables via the OOP API

When using the **OOP API directly** (i.e., calling methods on a `LifeTable` instance),
you can supply a per-policy table for the secondary or additional lives without replacing
the principal life's table.  This is useful when the first life is homogeneous (e.g., all
male, same cohort) but the remaining lives vary across policies.

**Two-life example** — shared table for *x*, per-policy table for *y*:

```python
from lactuca import LifeTable, config

lt_m = LifeTable("PER2020_Ind_1o", "m", cohort=1960, interest_rate=0.03)

# Zip mode (cartesian=False, default): broadcast sex="f" across 3 cohorts → 3 instances
lt_ys = LifeTable("PER2020_Ind_1o", "f", cohort=[1958, 1962, 1955], interest_rate=0.03)

x_ages = [60.0, 62.0, 65.0]
y_ages = [55.0, 58.0, 61.0]

config.decimals.annuities = 4
# table_y accepts the tuple returned by the vectorial constructor directly
result = lt_m.axy(
    (x_ages, y_ages),
    table_y=lt_ys,  # per-policy tables for y (tuple of 3 LifeTable instances)
    n=20,
)
print(result)
# [13.7232 13.5776 13.2123]
config.reset_to_defaults()
```

**Three-life example** — shared table for *x*, per-policy tables for *y* and *z*:

```python
from lactuca import LifeTable, config

lt_m = LifeTable("PER2020_Ind_1o", "m", cohort=1960, interest_rate=0.03)

# Zip mode (cartesian=False, default): broadcast sex across per-policy cohorts → tuples
lt_ys = LifeTable("PER2020_Ind_1o", "f", cohort=[1958, 1962], interest_rate=0.03)
lt_zs = LifeTable("PER2020_Ind_1o", "m", cohort=[1955, 1965], interest_rate=0.03)

x_ages = [60.0, 62.0]
y_ages = [55.0, 58.0]
z_ages = [50.0, 52.0]

config.decimals.annuities = 4
# zip(lt_ys, lt_zs) pairs per-policy y and z tables; list() makes it subscriptable
result = lt_m.axyz(
    (x_ages, y_ages, z_ages),
    tables_yz=list(zip(lt_ys, lt_zs)),  # [(lt_y0, lt_z0), (lt_y1, lt_z1)]
    n=20,
)
print(result)
# [13.2803 13.1642]
config.reset_to_defaults()
```

**N-life example** — shared table for *x*, per-policy tables for all other lives:

```python
from lactuca import LifeTable, config

lt_m = LifeTable("PER2020_Ind_1o", "m", cohort=1960, interest_rate=0.03)

# Zip mode (cartesian=False, default): broadcast sex="f" across per-policy cohorts → tuple
lt_ys = LifeTable("PER2020_Ind_1o", "f", cohort=[1958, 1962, 1955], interest_rate=0.03)

x_ages = [60.0, 62.0, 65.0]
y_ages = [55.0, 58.0, 61.0]

config.decimals.annuities = 4
# tables_others expects [[table_y_i], …] — wrap each instance in a 1-element list
result = lt_m.ajoint(
    [x_ages, y_ages],
    tables_others=[[lt_y] for lt_y in lt_ys],  # [[lt_y0], [lt_y1], [lt_y2]]
    n=20,
)
print(result)
# [13.7232 13.5776 13.2123]
config.reset_to_defaults()
```

:::{note}
The same syntax works for `äxy`/`äxyz`/`äjoint` (annuity-due), `Axy`/`Axyz`/`Afirst`
(insurance), and `nExy`/`nExyz`/`nEjoint` (pure endowment).  In each case:

- `table_y` (2-life): `None` | `LifeTable` (shared) | `list[LifeTable]` of length N.
- `tables_yz` (3-life): `None` | `[LifeTable, LifeTable]` (shared) | `list[[ty, tz], …]` of length N.
- `tables_others` (n-life): `None` | `list[LifeTable]` of length n_lives-1 (shared)
  | `list[list[LifeTable]]` of length N.

A `ValueError` is raised if the supplied list length does not match the number of
policies N.
:::

**Param-triggered batch** also applies to joint-life methods: keep the ages scalar and
pass any of `n`, `ts`, `d`, `ir`, `gr`, or `m` as an array of length N.

```python
from lactuca import LifeTable, config

lt_m, lt_f = LifeTable("PER2020_Ind_1o", ["m", "f"], cohort=1960, interest_rate=0.03)
lt = lt_m  # single-table OOP API

result = lt.axy([65, 60], n=[10.0, 20.0, 30.0])          # n sole trigger → NDArray (3,)
result = lt.axy([65, 60], m=[1, 6, 12])                   # m sole trigger → NDArray (3,)
result = lt.axy([65, 60], n=10, ir=[0.03, 0.04, 0.05])    # ir sole trigger → NDArray (3,)
result = lt.ajoint([65, 60, 55], n=[10, 20], ir=[0.03, 0.04])  # → NDArray (2,)
```

:::{note}
For `nExy`, `nExyz`, and `nEjoint` (pure endowments), only `n`, `ts`, and `ir` act as
param triggers — `d`, `gr`, and `m` are not applicable to these methods.
:::

---

## Aggregate portfolio flows

`return_flows=True` with an array of ages returns **aggregated expected cash flows** for
the portfolio, which are then used to compute present value measures such as BEL and
PVDBO.

### Structure of the returned dict

```python
from lactuca import LifeTable, config

lt   = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)
ages = [55, 60, 65, 70]

flows = lt.Ax(ages, n=6, ir=0.03, return_flows=True)

print(flows["time_grid"])    # [0.5 1.5 2.5 3.5 4.5 5.5]  — payment times (union grid, mid-year placement)
print(flows["expected_cf"])  # expected death benefits at each time point (undiscounted)
print(flows["pv_cf"])        # present-value contributions at each time point
print(flows["total_pv"])     # total APV across all 4 lives (sum of pv_cf)
# 0.19489468616630629
```

| Key | Type | Description |
|---|---|---|
| `time_grid` | `NDArray[float64]` | All distinct payment times (union grid of all policies) |
| `expected_cf` | `NDArray[float64]` | Expected cash flows **without discounting** at each `time_grid` point |
| `pv_cf` | `NDArray[float64]` | Present-value contributions at each `time_grid` point (discounted with `ir`) |
| `total_pv` | `float` | `np.sum(pv_cf)` — total present value of all portfolio flows at the discount rate `ir` |

:::{note}
With a **scalar** `x` (or scalar `ages[0]` for joint-life methods), `return_flows=True`
returns the per-payment engine dict documented in {doc}`inspecting_cashflows`.
:::

### Joint-life and *n*-life batch flows

`return_flows=True` is supported for all joint-life and *n*-life **method families** (two-life,
three-life, and *n*-life), with the same precision-mode requirement as single-life batch.
This includes two-life
(`axy`, `äxy`, `Axy`, `nExy`), three-life (`axyz`, `äxyz`, `Axyz`, `nExyz`), and *n*-life
(`ajoint`, `äjoint`, `Afirst`, `nEjoint`) variants.  The returned dict has the same
structure as for single-life batch:

```python
from lactuca import axy, LifeTable, config

# Zip mode (cartesian=False, default): pairs sex[i] with cohort[i] → 2 instances
lt_m, lt_f = LifeTable("PER2020_Ind_1o", ["m", "f"], cohort=[1960, 1963], interest_rate=0.03)

x = [55, 60, 65]   # ages of main pensioner (life x)
y = [50, 55, 60]   # ages of spouse / second life (life y)

# Annual pension amounts (€/year) — one per couple.
# axy() computes the joint-life annuity per unit of annual benefit;
# benefits= scales each policy's contribution to the aggregate flows
# by its actual annual pension amount.
annual_pensions = [50_000, 75_000, 20_000]

# Monthly joint-life pension annuity (m=12): PVDBO weighted by annual pension
flows = axy([lt_m, lt_f], (x, y), n=20, m=12, ir=0.03,
            return_flows=True, benefits=annual_pensions)

print(flows["total_pv"])     # portfolio PVDBO — joint pension BEL weighted by annual pension
print(flows["expected_cf"])  # aggregate expected monthly cash flows at each time_grid point
config.reset_to_defaults()
```

:::{important}
`return_flows=True` requires `calculation_mode='discrete_precision'` or
`calculation_mode='continuous_precision'` for all joint-life and *n*-life methods.
Using `discrete_simplified` or `continuous_simplified` raises `ValueError`.
:::

### Portfolio BEL with `benefits=` and aggregate flows (IFRS 17 / Solvency II)

`return_flows=True` combined with `benefits=sums` produces portfolio-level aggregate
cash flows scaled by per-policy benefit amounts — the natural building block for BEL,
PVDBO, and IFRS 17 / Solvency II valuations.

The examples below use the EIOPA risk-free curve as a concrete discount rate, but the
pattern applies to any `ir=` value.

**Approach A — per-policy APV, then dot-product (simplest)**

Compute unit APVs in a single batch call, then weight by benefit amount:

```python
from lactuca import LifeTable, InterestRate
import numpy as np

lt = LifeTable("PASEM2020_Gen_2o", "m", interest_rate=0.03)

ages  = [55, 60, 65, 70]
sums  = [100_000, 150_000, 200_000, 80_000]

# Replace 0.0342 with the actual EIOPA risk-free spot rate
eiopa_curve = InterestRate(0.0342)

unit_apv_eiopa = lt.Ax(ages, n=20, ir=eiopa_curve)      # unit APV at EIOPA rate, shape (4,)
bel_eiopa      = np.dot(unit_apv_eiopa, sums)     # portfolio BEL under IFRS 17 / Solvency II
print(f"BEL = {bel_eiopa:.2f}")
# BEL = 106953.84
```

**Approach B — aggregate expected cash flows via `return_flows=True` + `benefits=`**

Use `return_flows=True` together with `benefits=sums` to obtain the portfolio BEL in
`flows["total_pv"]` alongside the time-bucketed aggregate cash flows — all in a single
call.  This approach also gives access to the full cash-flow timeline, which is required
for IFRS 17 GMM / CSM calculations or ALM analysis.

The result dict exposes three equivalent paths to the same BEL figure:

- **`flows["total_pv"]`** — scalar sum pre-computed by the engine.
- **`np.sum(flows["pv_cf"])`** — explicit sum of the per-bucket discounted cash flows
  (`pv_cf[k]` already includes the discount factor, mortality weight, and mortality-placement offset).
- **`np.dot(flows["expected_cf"], ir.vn(t_grid + offset))`** — manual re-discounting of
  the undiscounted expected cash flows `expected_cf[k]` at the *correct* discount time
  `t_grid[k] + offset`.  For `Ax` with `mortality_placement='mid'` (default) and annual
  payments ($m = 1$) the offset is $0.5/m = 0.5$; deaths are assumed to occur at the
  midpoint of each annual interval.

All three are exact (within float64 round-off) and produce an identical result:

```python
from lactuca import Config, LifeTable, InterestRate
import numpy as np

cfg = Config()
cfg.mortality_placement = "mid"   # ensure mid-period placement for this example

lt = LifeTable("PASEM2020_Gen_2o", "m", interest_rate=0.03)

ages = [55, 60, 65, 70]
sums = [100_000, 150_000, 200_000, 80_000]

eiopa_curve = InterestRate(0.0342)

# Single call: portfolio BEL + time-bucketed aggregate cash flows
flows = lt.Ax(ages, n=20, ir=eiopa_curve, return_flows=True, benefits=sums)

# Way 1: pre-computed scalar — identical to Approach A
print(f"BEL via total_pv                 = {flows['total_pv']:.2f}")
# BEL via total_pv                 = 106953.84

# Way 2: explicit sum of per-bucket discounted cash flows
print(f"BEL via sum(pv_cf)               = {np.sum(flows['pv_cf']):.2f}")
# BEL via sum(pv_cf)               = 106953.84

# Way 3: re-discount expected_cf at time_grid + offset
# offset = {"beginning": 0.0, "mid": 0.5, "end": 1.0}[placement] / m
OFFSET_MAP = {"beginning": 0.0, "mid": 0.5, "end": 1.0}
m = 1  # annual payments
offset = OFFSET_MAP[cfg.mortality_placement] / m
vn = eiopa_curve.vn(flows["time_grid"] + offset)
print(f"BEL via dot(expected_cf, vn+off) = {np.dot(flows['expected_cf'], vn):.2f}")
# BEL via dot(expected_cf, vn+off) = 106953.84

cfg.reset_to_defaults()
```

:::{note}
**When to avoid `benefits=`: per-policy individual flows**

Use `benefits=` whenever you need a portfolio-level BEL or PVDBO — it is always more
efficient than looping.

The only reason to omit `benefits=` is when you need the **per-policy** flow breakdown
(not the portfolio aggregate) — for example to inspect each policy's cash-flow profile
individually.  In that case, call `return_flows=True` with a scalar age per policy and
reconstruct the portfolio aggregate manually:

```python
import numpy as np
from lactuca import LifeTable, InterestRate

lt   = LifeTable("PASEM2020_Gen_2o", "m", interest_rate=0.03)
ages = [55, 60, 65, 70]
sums = [100_000, 150_000, 200_000, 80_000]

# Per-policy flows — per_policy[i] is the per-payment dict for policy i
per_policy  = [lt.Ax(x, n=20, ir=0.03, return_flows=True) for x in ages]

# Reconstruct the portfolio aggregate
t_grid      = per_policy[0]["time_grid"]
all_cf      = np.array([f["expected_cf"] for f in per_policy])  # shape (N, T)
sums_arr    = np.asarray(sums, dtype=np.float64)                 # shape (N,)
weighted_cf = sums_arr @ all_cf                                  # shape (T,) — portfolio aggregate

# Discount at t_grid + offset (offset = 0.5 for mortality_placement='mid', m=1)
v_eiopa     = InterestRate(0.0342).vn(t_grid + 0.5)
bel_eiopa   = np.dot(weighted_cf, v_eiopa)
```
:::

:::{note}
`flows["total_pv"]` is already discounted at `ir` — the BEL in an insurance context, the
PVDBO in a pension / IAS 19 context, or the APV in any general present-value calculation.
No further discounting is needed.
:::

(payment-time-grid-options)=
### Payment-time grid options

`t_output=None` (default) returns the exact union of all payment times.
Pass an external `NDArray` as `t_output` to bucket the discounted flows onto a fixed
reporting grid (e.g., 20 annual IFRS 17 buckets, Solvency II projection ladder).
`total_pv` is **always exact** — `t_output` only reshapes `expected_cf` and `pv_cf`.

:::{dropdown} Advanced: t_output regimes and re-discounting guidance

When `t_output` is an external `NDArray`, the engine **buckets** the per-payment flows
onto the provided grid: each payment's already-discounted contribution is accumulated
into the nearest bucket via `np.searchsorted`, without any re-discounting.  The returned
`time_grid` matches the external grid exactly, making the output directly compatible
with regulatory reporting templates (e.g., Solvency II, IFRS 17).

| `t_output` | `time_grid` shape | `expected_cf` / `pv_cf` shape | `total_pv` |
|---|---|---|---|
| `None` (default) | One entry per distinct payment time | Same | Exact |
| External `NDArray` of length K | K bucket points | K entries — flows bucketed, not re-discounted | Exact — identical to default |

`total_pv` is always `np.sum(pv_cf)`.  Because each payment's present-value contribution
is discounted at its **exact** payment time before bucketing, the aggregate sum is
unaffected by the choice of output grid.  `t_output` controls only the **shape** of
`expected_cf` and `pv_cf` — not the total.

**Early payments vs. the first bucket.**  When bucketing onto an external grid, payment
times strictly **before** `t_output[0]` are accumulated into **bucket index 0** (left
clip).  Design reporting grids so the first bucket covers deferment and any in-advance
payments you need to isolate; otherwise early flows appear merged at the origin.

**What changes with `t_output`?**

The `expected_cf` and `pv_cf` arrays are condensed: instead of one entry per payment
time, each entry represents the aggregate of all payments within that bucket.
This is the pattern required when a regulatory template demands a fixed-bucket timeline
(e.g., 20 annual entries for a 20-year product with monthly payments).

The following example shows the reshape effect for a monthly life annuity (`äx`, `m=12`):

```python
from lactuca import LifeTable, config, payment_times

lt   = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)
ages = [55, 60, 65, 70]

config.decimals.annuities = 6

# Exact result (default): one entry per monthly payment time — 240 entries total
flows_exact = lt.äx(ages, n=20, m=12, ir=0.03, return_flows=True)
print(f"Exact  total_pv         : {flows_exact['total_pv']:.6f}")
print(f"Exact  len(expected_cf) : {len(flows_exact['expected_cf'])}")
print(f"Exact  time_grid[:4]    : {flows_exact['time_grid'][:4]}")
# Exact  total_pv         : 53.767205
# Exact  len(expected_cf) : 240
# Exact  time_grid[:4]    : [0.         0.08333333 0.16666667 0.25      ]

# Annual regulatory grid: condense 240 monthly entries into 20 annual buckets
annual_grid  = payment_times(n=20, m=1)
flows_annual = lt.äx(ages, n=20, m=12, ir=0.03,
                     t_output=annual_grid, return_flows=True)
print(f"Annual total_pv         : {flows_annual['total_pv']:.6f}")
print(f"Annual len(expected_cf) : {len(flows_annual['expected_cf'])}")
print(f"Annual time_grid        : {flows_annual['time_grid']}")
# Annual total_pv         : 53.767205   ← identical to exact
# Annual len(expected_cf) : 20
# Annual time_grid        : [ 1.  2.  3. … 20.]

config.reset_to_defaults()
```

`total_pv` is identical — `t_output` reshapes the cash-flow arrays without changing
the present value.

**Guidance for regulatory reporting**

- Use `t_output=None` (exact grid) for **internal pricing and model validation**, ALM
  duration analysis, and any use case requiring per-payment-time granularity.
- Use `t_output=<regulatory_grid>` to condense the output to the fixed set of bucket
  dates required by a reporting template (e.g., 20 annual IFRS 17 disclosure buckets,
  annual Solvency II SCR projection).
- `flows["total_pv"]` is exact in both cases — no approximation is introduced in the
  total present value.
- When working with the bucketed `expected_cf` values, be aware that each bucket
  aggregates payments from multiple exact payment times.  If you need to re-discount
  `expected_cf` at the bucket times for downstream calculations, prefer using `pv_cf`
  directly (already exactly discounted) to avoid introducing a timing mismatch.

:::{warning}
**IFRS 17: avoid re-discounting bucketed `expected_cf` at annual bucket times**

When `t_output` is an annual grid and `m > 1`, `expected_cf[k]` aggregates all
payments within year *k*.  Discounting this aggregate at the annual bucket time with
an external curve (e.g., the EIOPA risk-free curve) ignores intra-year timing — a
systematic error equivalent to assuming all monthly payments occur at year-end.

Two exact alternatives:

- Pass `ir=eiopa_curve` directly → `flows["total_pv"]` is already the exact
  EIOPA-discounted BEL, and `pv_cf` contains the per-bucket exact contributions.
- Or use `t_output=None` (exact 240-entry monthly grid for `m=12, n=20`) and
  discount at the exact payment times:
  `np.dot(flows["expected_cf"], eiopa_curve.vn(flows["time_grid"]))`.
:::

**Output grid regimes**

The relationship between `t_output` granularity and the product's payment frequency
defines three regimes with different practical and regulatory implications.

*Regime A — `t_output` coarser than payment times (recommended)*

`t_output` has fewer points than the exact payment grid (e.g., annual buckets for a
monthly product).  Each bucket accumulates several payments.  This is the canonical
regulatory use case: the bucket labels correspond to the reporting periods required by
the template, and each bucket sum represents the aggregate flow for that interval.

| Use case | Example `t_output` | Bucket meaning |
|---|---|---|
| IFRS 17 annual disclosure | `np.arange(1.0, n+1, 1.0)` | Year-end aggregate of all intra-year payments |
| Solvency II BEL ladder | `np.arange(1.0, n+1, 1.0)` | Annual flows for matching-adjustment |
| Quarterly ALM reporting | `np.arange(0.25, n+0.25, 0.25)` | Quarterly aggregate |

`total_pv` is exact because each payment is discounted at its actual payment time
before being placed in the bucket.

*Regime B — `t_output` finer than payment times (allowed, not recommended)*

`t_output` has more points than the exact payment grid (e.g., monthly `t_output` for
an annual product).  Most buckets receive zero flow; each annual payment falls on its
corresponding bucket point.

The result is valid but **sparse**: `total_pv` is exact, and `expected_cf` / `pv_cf`
are mostly zeros.  There is no information gain over `t_output=None`, and there is a
**float-precision risk**: grids built with `np.arange(1/12, n, 1/12)` accumulate IEEE
754 step error, so an integer-year payment time may not fall exactly on the intended
bucket point and can silently land in the adjacent bucket.  If a fine grid is required,
use `np.linspace` to avoid accumulated step error:

```python
# Safer: linspace distributes rounding error evenly
t_monthly = np.linspace(1/12, n, 12 * n)
```

For products with annual or lower payment frequency, `t_output=None` already returns a
compact exact grid with no risk of misalignment.  Prefer it over a finer `t_output`.

*Regime C — `t_output` with gaps that skip payment times (not recommended)*

`t_output` omits some intervals where payments occur (e.g., `t_output = [1.0, 3.0]`
for a product with payments at t=1, 2, 3).  The engine assigns each unmatched payment
to the last bucket whose label is ≤ its exact payment time (`searchsorted(side="right")
- 1`).  The payment at t=2.0 is silently absorbed into the t=1.0 bucket.

- `total_pv` remains exact — no present value is lost.
- `pv_cf[0]` (labelled t=1.0) contains the PV contribution of a payment that actually
  occurs at t=2.0.  For matching-adjustment, duration or SCR calculations, this creates
  a systematic timing mismatch proportional to the gap size.
- No exception or warning is raised.

:::{warning}
**Avoid Regime C for regulatory reporting.**  When constructing a custom `t_output`,
ensure it includes a bucket label for every distinct payment interval present in the
batch.  A payment that "falls back" into an earlier bucket will make `pv_cf` appear to
have a different timing profile than the actual cash flows, which can distort Solvency II
matching-adjustment eligibility tests and IFRS 17 liquidity disclosure tables.
:::

:::

### `return_flows` requires a precision calculation mode

:::{warning}
`return_flows=True` in batch mode is supported only in `discrete_precision` and
`continuous_precision`.  Using a `simplified` mode raises `ValueError`.
:::

| `calculation_mode` | `return_flows=True` in batch | Notes |
|---|---|---|
| `discrete_precision` | ✅ Supported | Default mode; full cash-flow decomposition available |
| `continuous_precision` | ✅ Supported | Integration-grid arrays; same dict keys: `time_grid`, `expected_cf`, `pv_cf`, `total_pv` |
| `discrete_simplified` | ❌ `ValueError` | Use `discrete_precision` for flow aggregation |
| `continuous_simplified` | ❌ `ValueError` | Use `discrete_precision` for flow aggregation |

---

(cohort-and-duration-instance-level-properties)=
## Cohort and duration: instance-level properties

`cohort` and `duration` are `LifeTable` constructor parameters — they cannot be
passed to actuarial functions as per-policy arguments.  To calculate a batch of
contracts (insurance policies, pension plan members, annuitants, etc.) with different
cohorts or select durations, use different `LifeTable` instances: one per distinct
value, or one shared instance updated via setters between groups.

All per-policy dimensions (`x`, `n`, `ir`, `m`, `d`, `ts`, `gr`, `benefits`, sex/table)
are covered in [Single-table batch](#single-table-batch) and [Multi-table batch](#multi-table-batch-functional-api).
The two dimensions that **cannot** be passed per-policy are:

| Dimension | Why it is a `LifeTable` param | How to handle a heterogeneous portfolio |
|---|---|---|
| `cohort` | Drives projection of the generational *qx* grid at construction time | `LifeTable(..., cohort=year)` or `lt.cohort = year`. Use one `LifeTable` per distinct cohort — see patterns below. |
| `duration` | Select-table offset applied at construction time | `LifeTable(..., duration=k)` or `lt.duration = k`. Use one `LifeTable` per distinct select duration. |

:::{note}
Passing lists to the `LifeTable` constructor (e.g.
`LifeTable("...", ["m", "f"], cohort=[1960, 1963])`) creates one instance per entry.
A `ResourceWarning` is emitted when more than 100 instances are created in a single
call (~300 KB of projected *qx* data is cached per generational table).

For portfolios where many policies share the same cohort, avoid the naive anti-pattern
of constructing a separate `LifeTable` per policy row
(`[LifeTable(..., cohort=c) for c in cohorts_array]` with N=300 000 entries would
create thousands of redundant instances).  Instead, build one `LifeTable` per
distinct cohort value and assemble a per-policy reference list — the lookup-dict
pattern shown below.
:::

### Few distinct cohorts (direct construction)

```python
from lactuca import LifeTable, ax, config

# Zip mode (cartesian=False, default): single sex "m", two cohorts → 2 instances
lt_1960, lt_1965 = LifeTable("PER2020_Ind_1o", "m", interest_rate=0.03, cohort=[1960, 1965])

ages   = [55, 60, 65]
tables = [lt_1960, lt_1965, lt_1960]   # one entry per policy
config.decimals.annuities = 4
result = ax(tables, ages, n=20)
print(result)
# [14.2416 14.1123 13.664 ]
config.reset_to_defaults()
```

### Large portfolios with many cohorts (lookup dict, no policy limit)

```python
from lactuca import LifeTable, TableKey, ax

# 300 000 contracts — cohorts spanning 1930–2000 (71 distinct values)
# (loaded from a DataFrame, CSV, database, etc.)
cohorts = [...]   # list of int birth years, length 300_000
ages    = [...]   # list of ages, length 300_000
sexes   = [...]   # 'm' or 'f' per member, length 300_000

# Step 1 — build one LifeTable per unique (sex, cohort) pair with return_dict=True (zip mode)
unique_pairs   = sorted(set(zip(sexes, cohorts)))
unique_sexes   = [s for s, c in unique_pairs]
unique_cohorts = [c for s, c in unique_pairs]
tables_by_key = LifeTable(
    "PER2020_Ind_1o", unique_sexes, cohort=unique_cohorts,
    return_dict=True, interest_rate=0.03,
)  # dict[TableKey, LifeTable]; emits ResourceWarning if > 100 instances

# Step 2 — assemble per-member table list and run a single batch call
table_list = [tables_by_key[TableKey("PER2020_Ind_1o", s, c)]
              for s, c in zip(sexes, cohorts)]
result = ax(table_list, ages, n=20)   # NDArray shape (300_000,), no Python loop
```

:::{tip}
When the unique `(sex, cohort)` pairs cover a **full Cartesian grid** (every sex with every
cohort value), skip pair extraction and use `cartesian=True` to build all combinations
directly — the result is the same `dict[TableKey, LifeTable]`:

```python
from lactuca import LifeTable, TableKey, ax

tables_by_key = LifeTable(
    "PER2020_Ind_1o", ["m", "f"],
    cohort=range(1930, 2001),
    cartesian=True,
    return_dict=True,
    interest_rate=0.03,
)  # 2 × 71 = 142 instances; emits ResourceWarning above 100

table_list = [tables_by_key[TableKey("PER2020_Ind_1o", s, c)]
              for s, c in zip(sexes, cohorts)]
result = ax(table_list, ages, n=20)
```
:::

Contracts sharing the same sex and cohort are calculated together — no per-member Python loop is needed.

### Memory-optimal: one table, group-then-update

When a portfolio spans many `(sex, cohort, duration)` combinations and memory is
constrained, a single `LifeTable` instance can be reused across all groups.
Assign the group's parameters via setters, then call the batch function for that group.
Only one table's *qx* projection is resident in memory at any time.

```python
import numpy as np
from lactuca import LifeTable, ax

# Portfolio data (300 000 members, mixed sex / cohort / duration)
# Plain lists — loaded from a DataFrame, CSV, database, etc.
sexes     = [...]   # 'm' or 'f' per member, length 300_000
cohorts   = [...]   # int birth year per member, length 300_000
durations = [...]   # int select duration (or None), length 300_000
ages      = np.array([...], dtype=np.float64)   # shape (300_000,)

# Sort indices by group key (sex, cohort, duration) to minimise setter calls
keys  = list(zip(sexes, cohorts, durations))
order = sorted(range(len(keys)), key=lambda i: keys[i])

result = np.empty(len(ages), dtype=np.float64)

# One reusable table — O(1) memory regardless of number of distinct groups
first = order[0]
lt = LifeTable("PER2020_Ind_1o", sexes[first], interest_rate=0.03,
               cohort=cohorts[first], duration=durations[first])

# Iterate over homogeneous groups
i = 0
while i < len(order):
    s_k, c_k, d_k = keys[order[i]]
    j = i
    while j < len(order) and keys[order[j]] == (s_k, c_k, d_k):
        j += 1
    group_idx  = order[i:j]
    group_ages = ages[group_idx]

    # Update only what changed (setters are idempotent — no-op if unchanged)
    if lt.sex != s_k:
        lt.sex = s_k
    if lt.cohort != c_k:
        lt.cohort = c_k          # recomputes qx projection for the new cohort
    if lt.duration != d_k:
        lt.duration = d_k

    result[group_idx] = ax(lt, group_ages, n=20)  # vectorised batch for this group
    i = j
```

**Comparison of the three patterns:**

| Pattern | Tables in memory | Best for |
|---|---|---|
| Direct construction (few cohorts) | K (one per unique cohort) | Families, scenarios (< ~10 groups) |
| Lookup dict | K (one per unique (sex, cohort) pair) | Large portfolios; multi-table batch dispatch handles grouping |
| One table + group-then-update | **1** | Extremely large portfolios or memory-constrained environments |

The lookup-dict pattern is simpler and preferred when memory is not the primary
concern: the multi-table dispatcher batches same-cohort policies automatically.
Use the one-table pattern only when the number of distinct groups is large enough
that K × 300 KB would be a meaningful constraint.

---

(pure-financial-annuities)=
## Pure financial annuities (`ir.a()`, `ir.ä()`)

{class}`lactuca.InterestRate` provides **mortality-free** batch annuities via `ir.a()` (annuity-immediate)
and `ir.ä()` (annuity-due).  These compute the present value of guaranteed fixed payment
streams with no mortality uncertainty — useful for fixed-income ALM, IFRS 17 guaranteed
elements, and duration analysis.

:::{note}
`ir.ä()` does not accept `cashflow_times`.  When payment timing is fully specified by
the caller, use `ir.a()` — the due/immediate distinction is only meaningful for
schedule-generated payments (i.e., when `n` and `m` define the schedule).
:::

### When to use

| Scenario | Method |
|---|---|
| PV of a guaranteed payment stream (ALM, liability matching) | `ir.a()` or `ir.ä()` |
| Annuity price curve (PV as a function of maturity) and duration analysis | `ir.a(n=maturities_arr)` |
| Per-policy annuity certain (different terms, deferments, or shifts) | `ir.a(n=n_arr, d=d_arr, ts=ts_arr)` |
| IFRS 17 guaranteed element BEL (no mortality) | `ir.a(n=n_arr, return_flows=True)` |
| Benefit-weighted bond/loan portfolio cashflows | `ir.a(n=n_arr, benefits=face_values)` (any `return_flows`) |
| Benefit-weighted pension portfolio cashflows (mixed in-payment / deferred) | `ir.ä(n=n_arr, d=d_arr, benefits=annual_pensions)` (any `return_flows`) |
| Level annual payment that funds a unit-PV liability (capital recovery) | `1.0 / ir.ä(n=n_arr)` |
| Per-policy payment frequency (mixed `m`) | `ir.a(n=n_arr, m=m_arr)` |
| Per-policy growth rate (inflation-linked / mixed `gr`) | `ir.a(n=n_arr, gr=gr_list)` |
| Project flows onto a reporting grid (IFRS 17 / ALM dates) | `ir.a(n=n_arr, return_flows=True, t_output=dates_arr)` |

### Batch detection

Batch mode is triggered automatically when **any** of `n`, `d`, `ts`, `m`, or `gr` is
a list, tuple, or `NDArray`:

| Input | Return type |
|---|---|
| All of `n`, `d`, `ts`, `m`, `gr` are scalar (or `None`) | `float` |
| `list`, `tuple`, or `NDArray` (any length ≥ 1) | `NDArray[np.float64]` shape `(N,)` |
| Pandas or Polars `Series` for any of `n`, `d`, `ts`, `m`, `gr`, `benefits`, `record_ids`, `t_output` | Same as NDArray — no `.to_numpy()` needed |
| Any array + `return_flows=True` | `dict` with keys `{time_grid, expected_cf, pv_cf, total_pv}` |
| Any array + `benefits=` | `NDArray[np.float64]` shape `(N,)` — per-record scaled PVs |
| Any array + `benefits=` + `return_flows=True` | `dict` with benefit-weighted aggregate portfolio flows |
| Any array + `on_error='nan'` | `BatchResult(values, errors)` |
| `m` or `gr` as sole trigger (scalar `n`, `d`, `ts`) | `NDArray[np.float64]` shape `(N,)` |
| Per-policy `m` or `gr` + `return_flows=True` | ❌ `ValueError` |

The batch size is determined by the longest array among `n`, `d`, `ts`, `m`, and `gr`.
Uniform sequences (e.g., `m=[2, 2, 2]`) are treated as scalar and dispatched efficiently.

### Broadcasting rules

Broadcasting follows the same length-1 rule as the single-table API: a scalar or
length-1 array is broadcast against the longest array.  Incompatible non-unit lengths
raise `ValueError`.  The rule applies to **all five** batch parameters (`n`, `d`, `ts`,
`m`, `gr`) simultaneously — the batch size is the length of the longest non-unit array.

| `n` length | `d` length | `ts` length | Result length |
|---|---|---|---|
| N | 1 | 1 | N |
| 1 | N | 1 | N |
| N | N | N | N |
| N | M (M ≠ N, M ≠ 1) | any | `ValueError` |

`m` and `gr` participate in the same broadcast logic: a scalar or length-1 value
broadcasts against the batch size determined by `n`, `d`, and `ts`; a length-N array
must match that size.

### Examples

**Annuity-certain PV curves over a range of maturities:**

```python
from lactuca import InterestRate, payment_times

ir = InterestRate(0.03)

maturities  = payment_times(n=30, m=1)   # [1.0, 2.0, …, 30.0]
a_curve     = ir.a(n=maturities)                   # NDArray shape (30,)
a_due_curve = ir.ä(n=maturities)                   # annuity-due curve
print(a_curve[:5])
# [0.97087379 1.9134697  2.82861135 3.7170984  4.57970719]
print(a_due_curve[:5])
# [1.         1.97087379 2.9134697  3.82861135 4.7170984 ]
```

**Per-policy terms (heterogeneous portfolio):**

```python
from lactuca import InterestRate

ir = InterestRate(0.03)

# Five policies with different maturities
n_arr = [10.0, 15.0, 20.0, 25.0, 30.0]
pv    = ir.a(n=n_arr)       # NDArray shape (5,)
print(pv)
# [ 8.53020284 11.93794003 14.87747486 17.41314754 19.60044135]
```

**Per-policy deferment (deferred guaranteed pensions):**

```python
from lactuca import InterestRate

ir = InterestRate(0.03)

# Same maturity, different deferment periods
pv = ir.a(n=20.0, d=[0.0, 5.0, 10.0])
print(pv)
# [14.87747486 12.8334405  11.07023851]
```

**Level annual payment that funds a unit-PV liability (capital recovery):**

```python
from lactuca import InterestRate

ir = InterestRate(0.03)

n_arr         = [10.0, 20.0, 30.0]
annuity_due   = ir.ä(n=n_arr)
net_premium   = 1.0 / annuity_due   # capital recovery factor: P · ä_n = 1  →  P = 1 / ä_n
print(net_premium)
# [0.11381603 0.06525797 0.04953326]
```

**Aggregate portfolio cash flows (IFRS 17 guaranteed element / ALM):**

```python
from lactuca import InterestRate

ir = InterestRate(0.03)

# Portfolio of 4 guaranteed annuity products (no mortality)
n_portfolio = [10.0, 15.0, 20.0, 25.0]
flows       = ir.a(n=n_portfolio, return_flows=True)

print(sorted(flows.keys()))
# ['expected_cf', 'pv_cf', 'time_grid', 'total_pv']

# flows['time_grid']   → NDArray of all payment times (union grid)
# flows['expected_cf'] → undiscounted cash flows (1/m per period, no mortality weighting)
# flows['pv_cf']       → discounted cash flows
# flows['total_pv']    → portfolio total PV = np.sum(flows['pv_cf'])
```

**`on_error='nan'` — robust batch processing:**

```python
from lactuca import InterestRate

ir = InterestRate(0.03)

n_with_invalid = [-5.0, 10.0, 20.0]   # first entry is invalid
result = ir.a(n=n_with_invalid, on_error='nan')

print(result.values)       # [nan, 8.53020284, 14.87747486]
print(result.errors.n_errors)  # 1
```

(benefit-weighted-portfolio-cash-flows)=
### Benefit-weighted portfolio cash flows

The optional `benefits` parameter scales each policy's PV by its benefit amount.
With `return_flows=False` (default) it returns per-policy scaled PVs:

$$\text{values}_i = \mathrm{round}\bigl(a_{\overline{n_i}|}^{(m)},\, d\bigr) \times b_i$$

where $a_{\overline{n_i}|}^{(m)}$ is the **unit** present value (same as calling the
method without `benefits=`), $d$ is the configured decimal precision for that product
(annuities, insurances, endowments), and $b_i$ is the benefit weight.  Rounding is
applied to the unit PV **before** multiplying by $b_i$ — not
$\mathrm{round}(a \times b_i, d)$.

For each policy $i$, the batch result equals the scalar path computed record by
record: `values[i] == f(...)[i] * b[i]` with **exact** equality (zero tolerance), the
same as looping over policies and calling the method once per row without `benefits=`,
then multiplying by `benefits[i]`.  This holds in all four calculation modes.

With `return_flows=True` it aggregates them into portfolio-level cash flows:

$$\text{total_pv} = \sum_i b_i \cdot a_{\overline{n_i}|}^{(m)}$$

This enables pricing heterogeneous portfolios where policies have different
face values, pension amounts, or capital amounts — whether you need per-policy PVs
or aggregate portfolio cash flows.

**Requirements and compatibility:**
- `benefits` requires batch mode (array `n`, `d`, `ts`, `m`, or `gr`)
- Compatible with `return_flows=False`: returns `NDArray[float64]` of scaled per-policy PVs
- Compatible with `return_flows=True`: returns aggregate portfolio flows dict
- Compatible with `on_error='nan'`: invalid entries produce `NaN`; `NaN × benefit = NaN`
- `on_error='nan'` and `return_flows=True` cannot be combined in the same call
- All benefit values must be finite and non-negative
- Supported for **all four** calculation modes (unlike `LifeTable.ax`, which requires a precision mode — `discrete_precision` or `continuous_precision` — for `return_flows=True`)

**Bond/loan portfolio aggregate cashflows:**

```python
from lactuca import InterestRate

ir = InterestRate(0.03)

# Five bonds with different maturities and face values
n_arr       = [5.0, 10.0, 10.0, 15.0, 20.0]
face_values = [100_000.0, 50_000.0, 75_000.0, 200_000.0, 150_000.0]

flows = ir.a(n=n_arr, benefits=face_values, return_flows=True)
print(f"Portfolio PV:  {flows['total_pv']:>15,.2f}")
# Portfolio PV:    6,143,454.32

# flows['time_grid']   → all payment dates (union grid)
# flows['expected_cf'] → undiscounted cashflows weighted by face values
# flows['pv_cf']       → discounted cashflows weighted by face values
# flows['total_pv']    → sum(face_values * unit_pv_per_bond)
```

**Pension portfolio aggregate cashflows (mixed in-payment and deferred):**

```python
from lactuca import InterestRate

ir = InterestRate(0.03)

# Mixed portfolio: first two pensioners already in payment (d=0),
# last two are active workers with deferred pensions (d=5 and d=10 years)
n_arr           = [15.0, 20.0, 25.0, 30.0]
d_arr           = [ 0.0,  0.0,  5.0, 10.0]
annual_pensions = [12_000.0, 18_000.0, 24_000.0, 15_000.0]

flows = ir.ä(n=n_arr, d=d_arr, benefits=annual_pensions, return_flows=True)
print(f"Portfolio BEL:  {flows['total_pv']:>15,.2f}")
# Portfolio BEL:   1,020,025.40
```

**Per-policy scaled PVs without aggregate flows (`return_flows=False`):**

```python
from lactuca import InterestRate

ir = InterestRate(0.03)

n_arr       = [5.0, 10.0, 10.0, 15.0, 20.0]
face_values = [100_000.0, 50_000.0, 75_000.0, 200_000.0, 150_000.0]

# returns NDArray[float64] shape (N,): each entry = face_value_i × unit_pv_i
scaled_pvs   = ir.a(n=n_arr, benefits=face_values)          # return_flows=False (default)
print(scaled_pvs)
# [457970.71... 426510.14... 639765.21... 2387587.01... 2231621.22...]
portfolio_pv = float(scaled_pvs.sum())                      # scalar portfolio PV
print(f"portfolio_pv = {portfolio_pv:,.2f}")
# portfolio_pv = 6,143,454.32
```

**Robust batch with invalid entries (`on_error='nan'`):**

```python
from lactuca import InterestRate

ir = InterestRate(0.03)

n_arr       = [-5.0, 10.0, 10.0, 15.0, 20.0]               # first entry is invalid
face_values = [100_000.0, 50_000.0, 75_000.0, 200_000.0, 150_000.0]

result = ir.a(n=n_arr, benefits=face_values, on_error="nan")
print(result.values)
# [             nan  426510.14...  639765.21...  2387587.01...  2231621.22...]
print(result.errors.n_errors)   # 1
```

**Manual equivalent (without `benefits=`):**

```python
import numpy as np
from lactuca import InterestRate

ir          = InterestRate(0.03)
n_arr       = [5.0, 10.0, 10.0, 15.0, 20.0]
face_values = [100_000.0, 50_000.0, 75_000.0, 200_000.0, 150_000.0]

# For scalar PV only — does NOT produce aggregate expected_cf / pv_cf arrays
unit_pvs     = ir.a(n=n_arr)                          # NDArray shape (N,)
portfolio_pv = float(np.dot(face_values, unit_pvs))   # scalar
print(f"portfolio_pv = {portfolio_pv:,.2f}")
# portfolio_pv = 6,143,454.32
```

:::{note}
**Difference from `LifeTable.ax`**

`ir.a()` and `ir.ä()` have no mortality: `expected_cf` reflects the pure payment schedule
(1/m per period) without multiplication by survival probabilities.  For mortality-discounted
annuities, use `lt.ax()` / `lt.äx()` on a `LifeTable` instance.

The `return_flows=True` dict schema is identical to `LifeTable.ax`: keys
`time_grid`, `expected_cf`, `pv_cf`, `total_pv`.
:::

---

## Performance notes

| Mode | Batch speedup vs scalar loop | Notes |
|---|---|---|
| `discrete_precision` (default) | **50–250×** | `return_flows=True` supported |
| `discrete_simplified` | **10–100×** | Woolhouse frequency adjustment; `return_flows=True` **not** supported — raises `ValueError` |
| `continuous_precision` | **10–100×** | Trapezoidal rule; `integration_steps` steps **per year** (default 200, so `ceil(n×200)` grid points for a term of `n` years); slower per policy than `discrete_precision`; `return_flows=True` supported |
| `continuous_simplified` | **10–100×** | Actuarial interpolation: averages annuity-due and annuity-immediate for the integer part, plus one-step fractional tail correction; `return_flows=True` **not** supported — raises `ValueError` |

The table above applies to **LifeTable** actuarial methods (`ax`, `äx`, `Ax`,
`nEx`).  All four modes support `return_flows=False` on those methods.
`InterestRate.a()` / `InterestRate.ä()` support batch `return_flows=True` in **all four**
modes — see {ref}`pure-financial-annuities`.

`discrete_precision` is the fastest option for batch annuities. `continuous_precision`
is slower per policy due to its finer numerical integration grid
(`integration_steps` steps per year, default 200).

Use `discrete_precision` for production batch runs; reserve `continuous_precision` for
integration-based fractional-age calculations. On **LifeTable**, `discrete_simplified` and
`continuous_simplified` do not support `return_flows=True` in batch mode.

---

## DataFrame workflow

Batch results are plain `NDArray[float64]` — they can be assigned directly as DataFrame
columns.  Lactuca methods accept Pandas and Polars Series (or any object with the
array protocol) directly as inputs for **all per-policy parameters** (`x`/`ages`, `n`,
`ts`, `d`, `ir`, `gr`, `m`, `benefits`, and `record_ids`), so no `.to_numpy()` conversion is needed.
In joint-life and *n*-life calls, each per-life age array within the `ages` tuple also
accepts a Series directly.  The shared parameters `cashflow_times`, `cashflow_amounts`,
and `t_output` accept Series too; they are converted to `NDArray[float64]` before use.

The following example passes Pandas DataFrame columns directly without `.to_numpy()`:

```python
import pandas as pd
import numpy as np
from lactuca import LifeTable, config

lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)
config.decimals.annuities  = 4
config.decimals.insurances = 4

df = pd.DataFrame({
    "age":       [55, 60, 65, 70],
    "sum_insured": [100_000, 150_000, 200_000, 80_000],
})

# Pass Series directly — no .to_numpy() needed
df["annuity"]   = lt.äx(df["age"], n=20)              # annuity-due (P = Ax / äx)
df["insurance"] = lt.Ax(df["age"], n=20) * df["sum_insured"]  # unit APV × sum insured
df["premium"]   = np.round(df["insurance"] / df["annuity"], 2)
print(df.to_string())
#    age  sum_insured  annuity  insurance  premium
# 0   55       100000  14.5608    11310.0   776.74
# 1   60       150000  14.1829    25695.0  1811.69
# 2   65       200000  13.5829    52620.0  3873.99
# 3   70        80000  12.6090    31752.0  2518.20

config.reset_to_defaults()
```

### Per-policy parameters as Series

Any per-policy parameter — not just ages — can be a Pandas or Polars Series.
This lets you pass DataFrame columns directly without manual conversion:

```python
import pandas as pd
import numpy as np
from lactuca import LifeTable, config

lt = LifeTable("PER2020_Ind_1o", "m", cohort=1960)
config.decimals.annuities = 4

df = pd.DataFrame({
    "age":       [55, 60, 65, 70],
    "term":      [30, 20, 15, 10],
    "tech_rate": [0.02, 0.03, 0.03, 0.04],
    "freq":      [1, 4, 12, 12],
    "indexing":  [0.00, 0.01, 0.02, 0.0],
})

# Pass every column as a Series — no .to_numpy() needed
df["annuity"] = lt.ax(
    df["age"],
    n=df["term"],
    ir=df["tech_rate"],
    m=df["freq"],
    gr=df["indexing"],
)
print(df.to_string())
#    age  term  tech_rate  freq  indexing  annuity
# 0   55    30       0.02     1      0.00  22.9617
# 1   60    20       0.03     4      0.01  16.4461
# 2   65    15       0.03    12      0.02  14.6891
# 3   70    10       0.04    12      0.00   8.1254

config.reset_to_defaults()
```

The same works with Polars:

```python
import polars as pl
from lactuca import LifeTable, config

lt = LifeTable("PER2020_Ind_1o", "m", cohort=1960)
config.decimals.annuities = 4

df = pl.DataFrame({
    "age":       [55, 60, 65, 70],
    "term":      [30.0, 20.0, 15.0, 10.0],
    "tech_rate": [0.02, 0.03, 0.03, 0.04],
    "freq":      [1, 4, 12, 12],
    "indexing":  [0.00, 0.01, 0.02, 0.0],
})

annuity = lt.ax(
    df["age"],
    n=df["term"],
    ir=df["tech_rate"],
    m=df["freq"],
    gr=df["indexing"],
)
df = df.with_columns(pl.Series("annuity", annuity))
print(df)

config.reset_to_defaults()
```

:::{note}
`InterestRate.a()` and `InterestRate.ä()` (pure financial annuities, no mortality)
also accept Pandas/Polars Series for `n`, `d`, `ts`, `m`, `gr`, `benefits`, `record_ids`,
and `t_output` — all without `.to_numpy()`.  A numeric `gr` Series is converted to
per-policy floats; an object-dtype Series of `GrowthRate` instances is treated like a list.
:::

---

## See also

- {doc}`inspecting_cashflows` — single-policy flow inspection and dict key reference
- {doc}`joint_life_calculations` — joint-life and multi-life methods
- {doc}`functional_api` — functional-style multi-table dispatch
- {doc}`calculation_modes` — `discrete_*` vs `continuous_*` modes explained
- {doc}`numerical_precision` — float64 precision policy and rounding

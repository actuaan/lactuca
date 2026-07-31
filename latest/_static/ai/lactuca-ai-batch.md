# Lactuca AI context — batch module

> Attach with `lactuca-ai-core.md` for portfolio / batch work.

<!--
lactuca_ai_context: batch
compatible_with_docs: 0.1.11
docs_base_url: https://www.lactuca.io/latest/
license: CC-BY-4.0
language: en
audience: end_users
-->

## When batch mode activates

- `x` (or joint `ages`) as **list, tuple, or ndarray** — length N ≥ 1.
- Or batch-only kwargs: `benefits=`, `record_ids=`, `on_error=`, `t_output=`.
- Or heterogeneous **`m`**, **`n`**, **`ir`**, etc. as arrays (see batch guide).

Scalar `x` + none of the above → returns **`float`**.

## Batch-only parameters (scalar → ValueError)

| Parameter | Role |
|-----------|------|
| `record_ids` | Labels in `BatchErrorReport` |
| `on_error='nan'` | `BatchResult(values, errors)` instead of raise |
| `benefits=` | Per-policy sum insured; BEL / PVDBO scaling |
| `t_output=` | Requires `return_flows=True` (precision modes) |

## on_error

```python
from lactuca import äx, LifeTable

lt = LifeTable("GRMF95", "m", interest_rate=0.03)
result = lt.äx([50, 150, 65], n=20, on_error="nan", record_ids=["P1", "P2", "P3"])
values, report = result  # tuple unpack (usual)
# Or: result.values, result.errors
```

- `on_error='nan'` is **not** compatible with `return_flows=True` in batch.
- Scalar call with `on_error` → `ValueError`.

## benefits=

Preferred for portfolio present values:

```python
pvs = äx(lt, ages, n=20, m=12, benefits=sums_insured)
```

Aggregate cashflow dicts need `return_flows=True` and **precision** modes
(`discrete_precision` or `continuous_precision`).

## return_flows in batch

| Mode | Batch `return_flows=True` |
|------|---------------------------|
| `discrete_precision` | OK |
| `continuous_precision` | OK |
| `discrete_simplified` | **ValueError** |
| `continuous_simplified` | **ValueError** |

Portfolio aggregate dict (batch `return_flows=True`, multiple policies) — keys are
**`time_grid`**, **`expected_cf`**, **`pv_cf`**, **`total_pv`** (not `t_grid`).


## Pending tables in batch

A table created with `pending=True` that has not been finalized with `configure()`
raises `ValueError` immediately when passed to any batch entry point (`tables=` in
`ajoint`, `axy`, `Axy`, `axyz`, etc.). Configure before passing to batch.

For heterogeneous portfolios (different cohort/duration per policy), use
`TableRegistry` instead of a mutated shared instance:

```python
from lactuca import LifeTable, TableRegistry, ax

reg = TableRegistry(LifeTable)
tables  = [reg.get_or_create(None, "PER2020_Ind_1o", p["sex"], cohort=p["cohort"]) for p in policies]
results = ax(tables, [p["age"] for p in policies], ir=0.03)
```

`interest_rate` is not part of the cache key — apply on first `get_or_create` only;
use batch `ir=` for per-policy rates with the same demographic key. Pass a **concrete**
table class (`LifeTable`, `DisabilityTable`, `ExitTable`); abstract `DecrementTable`
raises `TypeError`. Call `reg.clear()` to empty the cache (`config.reset()` does not
clear user registries).

See: https://www.lactuca.io/latest/user_guide/using_tables.html#deferred-construction
API: https://www.lactuca.io/latest/api/table_registry.html

## Anti-patterns

```python
# WRONG: loop with cohort setter on shared table for many policies
for row in portfolio:
    lt.cohort = row.cohort  # last row wins if table reused incorrectly
    pv = lt.äx(row.age, n=row.n)

# BETTER: batch ages + functional API or vectorized cohort handling
```

Portfolio rows with birth dates — derive `x` with top-level **`alb`** / **`age_last_birthday`**
before batch `äx`; do not hand-roll year differences. See `lactuca-ai-core.md` § Date utilities.

```python
# WRONG: record_ids on scalar call
lt.äx(50, n=20, record_ids=["x"])  # ValueError
```

## Joint / n-life batch

```python
from lactuca import axy

pvs = axy([lt_m, lt_f], (x_arr, y_arr), n=20, m=12, ir=0.03)
```

- Joint-life annuities (`axy`, `äxy`, …) and first-death insurances (`Axy`, `Axyz`,
  `Afirst`) accept `m`, `d`, `gr` like single-life products.
- Per-policy tables: functional `table=[lt1, lt2, …]` with aligned arrays.

## Functional multi-table

```python
from lactuca import Ax

pvs = Ax(table=[lt_a, lt_b, lt_c], x=[45, 50, 55], n=20)
```

## Reference

https://www.lactuca.io/latest/user_guide/batch_calculations.html

For custom payment grids in portfolio examples, use top-level
`payment_times` / `tiered_amounts` (`api/utils.html`) — not `lactuca.functional`.
For entry ages from birth + valuation dates, use `alb` / `age_last_birthday`
(`api/dates.html`, `user_guide/dates_guide.html`).

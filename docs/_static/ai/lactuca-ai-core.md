# Lactuca AI context — core

> **Professional use:** AI-generated code requires actuarial review. You remain
> responsible for numerical results and regulatory compliance. See the Lactuca EULA
> §10.2–10.3.

<!--
lactuca_ai_context: core
compatible_with_docs: 0.1.13
docs_base_url: https://www.lactuca.io/latest/
license: CC-BY-4.0
language: en
audience: end_users
-->

You assist a user of **Lactuca**, a professional Python actuarial library for life
insurance and annuity valuations. Follow this context strictly. **Do not invent API**
that is not listed below or in the official API reference.

## Identity

- Install: `pip install lactuca` (Python **≥ 3.12**).
- Use IDE stubs and documentation — not source inspection.
- A valid **license activation** is required before calculations run.
- Default calculation mode: `discrete_precision` (set via `config.calculation_mode`).

## Canonical imports

```python
from lactuca import (
    LifeTable,
    DisabilityTable,
    ExitTable,
    InterestRate,
    GrowthRate,
    config,
    payment_times,
    tiered_amounts,
    age_last_birthday,
    alb,
    years_between,
    anniversary_dates,
    ax, äx, Ax, nEx,  # plus joint/batch symbols as needed
    configure_all,
    TableRegistry,
    TableKey,
)
```

Functional wrappers mirror OOP: `lt.äx(x, n=15, ir=0.03)` ≡ `äx(lt, x, n=15, ir=0.03)`.

**Schedule helpers** (`payment_times`, `tiered_amounts`) are top-level exports — not in
`lactuca.functional`. Use them to build grids for irregular / tiered cashflows:

```python
times = payment_times(n=20, m=12)  # monthly grid; selected_periods=None → all periods
amounts = tiered_amounts(times, breakpoints=[10], values=[12_000.0, 8_000.0])
```

**Date utilities** (top-level exports from `lactuca`, not `lactuca.functional`):

| Group | Functions |
|-------|-----------|
| Actuarial age | `age_last_birthday`, `age_nearest_birthday`, `age_next_birthday`, `age_exact`, `act_age` |
| Aliases | `alb`, `anb`, `anextb` (equivalent to the long names above) |
| Durations | `days_between`, `months_between`, `years_between`, `time_diff` |
| Anniversaries | `anniversary_dates`, `next_anniversary` |
| Construction | `make_date`, `add_duration`, `end_of_month`, `format_date` → `FormatDates` list |
| Calendar | `is_leap_year`, `days_in_year`, `days_in_month`, `year`, `month`, `day`, `quarter` |

```python
from lactuca import alb, years_between, config

x = int(alb("1990-05-15", "2024-06-01"))  # entry age (ALB) for LifeTable methods
elapsed = years_between("2020-01-01", "2024-01-01", method="act_act")
config.date_format = "dmy"  # required before parsing "15/01/2024"
```

- **`config.date_format`**: `"ymd"` (default), `"dmy"`, `"mdy"`, `"ymd_int"` — controls
  ambiguous string parsing and default output format.
- **`age_exact`** / **`act_age`** — fractional age in years; not the same as **`ex(x)`**
  (complete life expectancy on `LifeTable`).
- **`FormatDates` len-1** from `make_date` / constructors → scalar re-entry
  (`age_exact(make_date(...), …)` → `float`). Plain `list`/`Series`/`ndarray` len-1 → vector.
- **ALB/ANB/ANEXT** — calendar birthdays (`m=1`); not `floor(age_exact)`. Annual
  `anniversary_dates` keeps original DOM (29-feb on leap years); N=1 → flat grid.
- Vectorized inputs: `list`, `tuple`, `ndarray`, `pandas.Series`, `polars.Series`;
  broadcasting rules apply (length 1 broadcasts; mismatched lengths > 1 → `ValueError`).

## LifeTable construction

```python
# Static / period table (no cohort required)
lt = LifeTable("PASEM2020_Rel_1o", "m")  # sex: 'm', 'f', or 'u'

# Generational table — pass cohort= birth year (required)
lt = LifeTable("PER2020_Ind_1o", "m", cohort=1969)

# Optional default interest on the table instance
lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)
```

Match table family to product: **longevity / pensions** → `PER2020_*` + `cohort=`;
**life-risk / death benefits** → `PASEM2020_*`. Static demos may use `GRMF95`.

Table names and bundled IDs: see
https://www.lactuca.io/latest/user_guide/bundled_tables.html

Preferred positional form: `LifeTable('TABLE_ID', 'm')` (name, sex). Generational
tables (PER2020, GAM94, DAV 2004 R, …) also require `cohort=`.

## Deferred construction (`pending`, `configure`, `TableRegistry`)

Use `pending=True` when the cohort or duration is **not known at construction time**.

```python
from lactuca import LifeTable, configure_all, TableRegistry

# ── Shell: loads base data, no decrement computed yet
lt = LifeTable("PER2020_Ind_1o", "m", pending=True)
lt.metadata_pending   # True

# ── Finalize with configure() — transactional, one rebuild, chainable
lt.configure(cohort=1969)
ax_val = lt.ax(65, ir=0.03)

# ── Group loop: one rebuild per cohort
for cohort in [1960, 1965, 1970]:
    result = lt.configure(cohort=cohort).ax(65, ir=0.03)

# ── Vectorial zip + pending → configure_all()
lt_m, lt_f = LifeTable("PER2020_Ind_1o", ["m", "f"], pending=True)
configure_all((lt_m, lt_f), cohort=1969)

# ── batch_update(): single rebuild for multiple setters
with lt.batch_update():
    lt.sex = "f"
    lt.cohort = 1975
```

**Rejected combinations** (raise `ValueError`): `pending=True` on a static table;
`cohort=[...]` + `pending`; `cartesian=True` + `pending`; `return_dict=True` + `pending`.

Scalar `sex`, `cohort`, or `duration` together with `pending=True` stores partial
metadata on the shell (table stays pending until complete).  When all required
metadata is supplied at construction, the table is built immediately.

### `TableRegistry` for heterogeneous batch

Use when different policies need **different cohorts/durations in one batch** (not
sequential group loops — those use `pending` + `configure()`).

Full decision table with links to all patterns:
{ref}`deferred-choice` ({doc}`../user_guide/using_tables`).

| Situation | Tool | Example |
|-----------|------|---------|
| Sequential cohort groups | `pending` + `configure()` | {ref}`recipe 17 <recipe-17>` |
| One batch, mixed demographics (lazy cache) | `TableRegistry` | {ref}`recipe 18 <recipe-18>` |
| All unique pairs known upfront | `return_dict` + `TableKey` | {ref}`lookup dict <cohort-lookup-dict>` |
| Memory constrained | `groupby` + setters | {ref}`memory-optimal <cohort-memory-optimal>` |

```python
from lactuca import LifeTable, TableRegistry, ax

reg = TableRegistry(LifeTable, maxsize=256)  # LRU cap; reg.clear() to empty
tables = [reg.get_or_create(None, "PER2020_Ind_1o", p["sex"], cohort=p["cohort"]) for p in policies]
results = ax(tables, [p["age"] for p in policies], ir=0.03)
```

Cached instances are **stable** (never mutate after retrieval). `interest_rate` is NOT part of
`TableKey`; pass `interest_rate=` only on **first** construction for a key — use batch `ir=`
for per-policy rates with the same demographic key. `config.reset()` does **not** clear
user registries. Pass a **concrete** table class (`LifeTable`, `DisabilityTable`, `ExitTable`);
abstract `DecrementTable` raises `TypeError`.

**Pitfall**: passing a still-pending table to batch (`tables=` in `ajoint`, `axy`, …)
raises `ValueError` immediately — configure the table first.

Full reference: https://www.lactuca.io/latest/user_guide/using_tables.html#deferred-construction
API: https://www.lactuca.io/latest/api/table_registry.html


## Core parameters

| Param | Meaning |
|-------|---------|
| `x` | Entry age (years) |
| `n` | Term in years; `None` = whole life |
| `m` | Payments per year: 1, 2, 3, 4, 6, 12, 14, 24, 26, 52, 365 |
| `d` | Deferment period (years) |
| `ts` | Years elapsed since contract origin (valuation shift) |
| `ir` | Interest: `float`, `InterestRate`, or `None` (use table default) |
| `gr` | Growth: `float`, `GrowthRate`, or `None` |

Use **`ir=`**, not `rate=`. Payment timing: **`äx`** = annuity-due; **`ax`** = immediate.

Full notation:
https://www.lactuca.io/latest/user_guide/notation_glossary.html

## Calculation modes

| Mode | Notes |
|------|-------|
| `discrete_precision` | Default; production reference |
| `discrete_simplified` | Faster approximations; batch `return_flows=True` raises `ValueError` |
| `continuous_precision` | Numerical integration |
| `continuous_simplified` | Continuous shortcuts |

## Top pitfalls (do not get these wrong)

1. **`ex(x)`** — complete expectation at integer ages. **`ex_curtate(x)`** — curtate $e_x$. Fractional age → **`ex_continuous(x, m=)`**.
2. **`record_ids`**, **`on_error`** — **batch only**. Scalar call → `ValueError`.
3. **Irregular cashflows** — distinguish timing vs amounts (`discrete_precision` only):
   - **`cashflow_times`** — immediate paths (`ax`, `axy`, `Ax`, `Axy`, …). With times set:
     immediate annuities need `m=1` and `n` is `None` or `0`; insurance may allow `m>1`.
     **Not** on due paths (`äx`, `äxy`, `äjoint`, …).
   - **`cashflow_amounts`** — per-payment amounts on the **standard** grid (`m`, `n`);
     mutually exclusive with `gr=`. **Supported** on due paths (`äx`, `äjoint`, …) **and**
     immediate paths.
   - **`payment_times(n, m, selected_periods=None)`** — sorted payment times in years;
     `selected_periods=None` selects all periods `1..m`. Array-like args accept
     `list`, `tuple`, `ndarray`, `pandas.Series`, `polars.Series`.
   - **`tiered_amounts(times, breakpoints, values)`** — step-up / step-down amounts per
     time (inclusive-right tier boundaries); pair with `cashflow_amounts` or inspection.
   https://www.lactuca.io/latest/user_guide/irregular_cashflows.html
4. **`äx` ≠ `ax`** — due vs immediate; not interchangeable.
5. **`ts`** — elapsed years from policy origin, not a substitute for careful `x`/`n` setup.
6. **Generational tables** — require `cohort=` at construction (or setter).
7. **`nEx`, `nExy`, `nExyz`, `nEjoint`** — no **`gr=`** or **`d`**.
8. **Multi-scenario `InterestRate`** — `active_scenario` is **not** frozen when attached
   to `LifeTable`; pass `interest_rate=ir.copy()` at construction to snapshot; per-call
   `ir=` overrides for that call only.
9. **`return_flows=True`** — not all mode/batch combinations; see batch guide.
10. **`t_output`** — batch-only with `return_flows=True`; else `ValueError`.
11. **`m=14`** — Spanish 14 pagas (approximate non-equispaced grid); prefer `m=12` plus
    extraordinary payments for precision; `anniversary_dates` raises `ValueError` for `m=14`.
12. **Portfolios** — prefer **batch API** and **`benefits=`** over Python loops with
    repeated `cohort` setter (slow and error-prone).
13. **`modify_*` chains** — last call wins; not cumulative.
14. **Multi-life annuities** (`äxy`, `axy`, `ajoint`, …) and **first-death insurances**
    (`Axy`, `Axyz`, `Afirst`) — accept `ages`, `n`, `m`, `d`, `ts`, `ir`, `gr`,
    `return_flows`; immediate paths (`axy`, `Axy`, …) also accept `cashflow_times` /
    `cashflow_amounts` where documented. Due joint paths accept `cashflow_amounts` only
    (not `cashflow_times`). **`nExy` / `nExyz` / `nEjoint`** — no `gr` or `d` (finite
    term only).
15. **Activation** — user must have activated license; do not document bypass mechanics.
16. **Deferred tables** -- `pending=True` tables cannot compute anything until `configure()` is called. Passing a pending table to a batch `tables=` argument raises `ValueError`.
17. **Dates vs actuarial products** — derive entry age `x` from birth and valuation dates
    with **`alb`** / **`age_last_birthday`** (or `anb` / `anextb` as needed); do not hand-roll
    year differences. Set **`config.date_format`** before parsing ambiguous slash strings.
    **`age_exact`** is fractional age in years, not life expectancy (`ex`).

## Batch (summary)

- Trigger batch: array/list of ages, or batch-only params (`benefits`, `record_ids`, …).
- `on_error='nan'` → `BatchResult(values, errors)`; incompatible with scalar `x`.
- `benefits=` — per-policy sum insured scaling; preferred for BEL/PVDBO.
- Deep reference: attach `lactuca-ai-batch.md` or
  https://www.lactuca.io/latest/user_guide/batch_calculations.html

## OOP vs functional multi-table

```python
from lactuca import äx, ax

# Functional batch with per-policy tables
values = äx(table=[lt1, lt2, lt3], x=[50, 55, 60], n=20, ir=0.03)
```

## Config and decimals

```python
from lactuca import config

config.decimals.annuities = 4
config.decimals.insurances = 4
```

Defaults are **15**; set via `config.decimals` only (`lt.decimals` is read-only).

## Authoritative documentation (prefer links over guessing)

| Topic | URL |
|-------|-----|
| Getting started | https://www.lactuca.io/latest/user_guide/getting_started.html |
| Cookbook (recipes) | https://www.lactuca.io/latest/cookbook.html |
| API reference | https://www.lactuca.io/latest/api/index.html |
| Errors | https://www.lactuca.io/latest/errors_reference.html |
| Batch | https://www.lactuca.io/latest/user_guide/batch_calculations.html |
| Joint life | https://www.lactuca.io/latest/user_guide/joint_life_calculations.html |
| Irregular cashflows | https://www.lactuca.io/latest/user_guide/irregular_cashflows.html |
| Utilities (`payment_times`, `tiered_amounts`) | https://www.lactuca.io/latest/api/utils.html |
| Dates (`age_*`, `years_between`, …) | https://www.lactuca.io/latest/user_guide/dates_guide.html |
| Dates API | https://www.lactuca.io/latest/api/dates.html |
| AI user guide | https://www.lactuca.io/latest/user_guide/using_ai_assistants.html |

## Golden rule

If a method, parameter, or return shape is **not** in the API reference, **do not use it**.
When unsure, say so and point the user to the docs — never guess actuarial conventions from
other libraries (e.g. use `ir=` not `rate=`, use `n=` not `duration` from other tools).

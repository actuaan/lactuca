# Using Actuarial Tables

This page covers instantiation patterns and runtime control that extend beyond the basics.
For your first table, computing probabilities, and annuity examples, see {doc}`getting_started`.

## Quick recap: the three table classes

```python
from lactuca import LifeTable, DisabilityTable, ExitTable

lt = LifeTable("PASEM2020_Rel_1o", "m")       # mortality table
dt = DisabilityTable("DummySD2015", "m")      # disability incidence table
et = ExitTable("DummyEXIT", "m")              # exit / withdrawal table
```

Basic constructor parameters — `sex`, generational `cohort`, unisex blend, and first-look
inspection — are covered in {doc}`getting_started`.

### Generational improvement on disability and exit tables

When a bundled **generational** `DisabilityTable` or `ExitTable` is loaded (`cohort=` required),
Lactuca applies the **same improvement-formula dispatch** as for mortality tables
(`exponential_improvement`, `linear_improvement`, `discrete_improvement`,
`projected_improvement` — see {doc}`mortality_improvement`).  The engine projects
:math:`i_x` or :math:`o_x` along the cohort diagonal; it does **not** embed disability-
 or lapse-specific regulatory methodologies.  Validate projection assumptions against your
own tables and practice before reserving or pricing.

## Default interest rate

Pass `interest_rate` to avoid specifying it in every method call:

```python
from lactuca import LifeTable

lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)

ax = lt.ax(65, n=10)              # uses 3%
ax2 = lt.ax(65, n=10, ir=0.02)   # overrides to 2% for this call only
print(f"ax (3%): {ax:.4f}")
print(f"ax (2%): {ax2:.4f}")       # slightly higher — lower discount rate
```

To use a term-structure instead of a flat rate, construct an {class}`lactuca.InterestRate`
object and pass it either at construction time or afterwards:

```python
from lactuca import LifeTable, InterestRate

ir = InterestRate(terms=[10, 10], rates=[0.025, 0.035, 0.04])

# Option A — pass at construction
lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=ir)

# Option B — assign after construction (equivalent)
lt2 = LifeTable("PASEM2020_Rel_1o", "m")
lt2.interest_rate = ir
print(round(lt.ax(65, n=20), 4))   # same result from either option
```

Multi-scenario `InterestRate` containers can be passed as `interest_rate=` or assigned
via `lt.interest_rate = ir`.  `lt.interest_rate` holds a **reference** to the same
object — calculations without an explicit `ir=` use whichever scenario is active
**at each call**.  To freeze the scenario, pass `ir.copy()` or a simple sub-curve
from `ir.scenarios["base"]`.  See {ref}`interest-rate-scenarios-lifetable`.

## Vectorial creation

For small families or scenario analysis, pass a sequence for `sex`,
`cohort` (generational tables), or `duration` (select-ultimate tables).  Lactuca returns
a `tuple` of independent instances:

```python
from lactuca import LifeTable

# Two sexes, different cohorts: (male, 1960) and (female, 1963)
ltm, ltf = LifeTable("PER2020_Ind_2o", ("m", "f"), cohort=(1960, 1963))

# Same sex, two cohorts — scenario analysis
born60, born70 = LifeTable("PER2020_Ind_2o", "m", cohort=[1960, 1970])

# Select-ultimate: three duration slices at once
# AM92_AF92 uses the CMI Duration-0 convention; most other tables start at duration=1
d0, d1, d_ult = LifeTable("AM92_AF92", "m", duration=(0, 1, "ult"))

# Select-ultimate generational: two sexes at the same cohort and duration
lt_m, lt_f = LifeTable("DAV2004R_SelUlt_1o", ("m", "f"), cohort=1990, duration=1)
print(round(born60.tpx(40, 25), 6), round(born70.tpx(40, 25), 6))  # gen-cohort difference
```

Parameters are aligned **element-wise across all four** (`table_name`, `sex`, `cohort`,
`duration`), not as a Cartesian product.  Two table names and two sexes in zip mode produce
**2** instances — `table_name[0]` paired with `sex[0]`, and so on.  All four can vary
together:

```python
from lactuca import LifeTable

# Length-2 sequences → 2 instances:
# instance 0: (male,   cohort=1960, duration=1)
# instance 1: (female, cohort=1963, duration=2)
lt0, lt1 = LifeTable("DAV2004R_SelUlt_1o", ("m", "f"), cohort=(1960, 1963), duration=(1, 2))

# Multiple base tables, zip mode (element-wise pairing)
lt_ind, lt_col = LifeTable(["PER2020_Ind_1o", "PER2020_Col_2o"], "m", cohort=1960)
```

To create every combination (e.g. 2 tables × 2 sexes = 4 instances), pass
`cartesian=True` — see [Cartesian-product creation](#cartesian-product-creation) below.

(vectorial-interest-rate)=
### Default interest rate in vectorial creation

`interest_rate` can be passed directly in the constructor and will be assigned to every
vectorially created instance.  Pass a scalar to broadcast the same rate to all instances,
or a sequence of the same length to assign a distinct rate per instance:

```python
from lactuca import LifeTable, InterestRate

# Scalar broadcast — both instances get interest_rate=0.03
lt_m, lt_f = LifeTable("PASEM2020_Rel_1o", ("m", "f"), interest_rate=0.03)
print(lt_m.interest_rate)  # 3.00 %
print(lt_f.interest_rate)  # 3.00 %

# Per-instance sequence — different rates for each instance
lt_m, lt_f = LifeTable("PASEM2020_Rel_1o", ("m", "f"), interest_rate=[0.03, 0.035])
print(lt_m.interest_rate)  # 3.00 %
print(lt_f.interest_rate)  # 3.50 %

# InterestRate objects are also accepted (scalar or sequence)
ir = InterestRate(terms=[10], rates=[0.025, 0.04])
lt_m, lt_f = LifeTable("PASEM2020_Rel_1o", ("m", "f"), interest_rate=ir)
```

Passing `interest_rate=None` (the default) leaves all instances with no default rate,
which is identical to constructing them individually without the argument.

### Broadcast rules

| `table_name` length | `sex` length | `cohort` length | `duration` length | Instances created |
|---|---|---|---|---|
| 1 | 1 | 1 | 1 | 1 (plain scalar case) |
| N | 1 | 1 | 1 | N (other params replicated) |
| 1 | N | 1 | 1 | N (other params replicated) |
| 1 | 1 | N | 1 | N (other params replicated) |
| 1 | 1 | 1 | N | N (other params replicated) |
| N | N | N | N | N (one-to-one alignment) |

Any length other than 1 or N raises `ValueError`.  Each parameter can be passed as
a `tuple` or a `list` — both are accepted interchangeably.  Pass `return_dict=True` to
receive a `dict[`{class}`lactuca.TableKey`, `LifeTable`]` instead of a `tuple`.

:::{note}
For portfolios with **many distinct cohorts** (dozens to hundreds), it is more memory-efficient
to build a dict of unique tables in a plain loop and pass a per-policy list to the batch API —
see {ref}`bulk-portfolios` below and {doc}`batch_calculations`.
A `ResourceWarning` is emitted automatically when more than 100 instances are constructed in
a single vectorial call (~300 KB of projected *qx* data cached per generational table).
:::

(cartesian-product-creation)=
### Cartesian-product creation

Pass `cartesian=True` to generate every combination of (`table_name`, `sex`, `cohort`,
`duration`) in a single constructor call.  This is appropriate for **assumption sensitivity
grids**, **pricing studies**, and **regulatory stress scenarios** where every combination is
needed regardless of whether any policy occupies it.

```python
from lactuca import LifeTable, TableKey

# 2 tables × 2 sexes × 1 cohort = 4 instances
tables = LifeTable(
    ["PER2020_Ind_1o", "PER2020_Col_2o"],
    ["m", "f"],
    cohort=1960,
    cartesian=True,
    return_dict=True,   # returns dict[TableKey, LifeTable]
)

# Access by structured key — no positional guessing
lt = tables[TableKey("PER2020_Ind_1o", "m", 1960)]
print(round(lt.äx(65, ir=0.03), 4))

# Pricing grid: 2 tables × 2 sexes × 71 cohorts = 284 instances
grid = LifeTable(
    ["PER2020_Ind_1o", "PER2020_Col_2o"],
    ["m", "f"],
    cohort=range(1930, 2001),
    cartesian=True,
    return_dict=True,
)
print(len(grid))   # 284
```

:::{important}
**Cartesian mode is for study grids, not portfolio processing.**
For a real insurance portfolio use the **groupby pattern** instead: group policies by
`(table_name, sex, cohort, duration)`, create one `LifeTable` per group, and pass the
group's ages array to the batch API.  `cartesian=True` creates every combination
regardless of whether any policy occupies it — wasted instances for sparse portfolios.
:::

#### `TableKey`: structured lookup keys

{class}`lactuca.TableKey` is a `NamedTuple` with five fields:

```python
from lactuca import TableKey

TableKey("PER2020_Ind_1o", "m")              # cohort=None, duration=None, unisex_blend=None
TableKey("PER2020_Ind_1o", "m", 1960)        # explicit cohort; duration=None
TableKey("PER2020_Ind_1o", "m", 1960, None)  # explicit duration
TableKey("PASEM2010", "u", unisex_blend=0.55) # cohort=None, duration=None
```

| Field | Type | Notes |
|---|---|---|
| `table_name` | `str` | Required — table identifier |
| `sex` | `str` | Required — `'m'`, `'f'`, or `'u'` |
| `cohort` | `int` or `None` | `None` for period (non-generational) tables |
| `duration` | `int`, `str`, or `None` | `None` for aggregate (non-select) tables |
| `unisex_blend` | `float` or `None` | Non-`None` only when `sex='u'` |

:::{warning}
**Float precision caveat for `unisex_blend`.**
Always use the **same literal float** that was passed at construction time.
`TableKey(..., unisex_blend=0.55)` requires exactly `0.55`, not `0.5 + 0.05`
(which may differ by one ULP due to IEEE 754 rounding).
:::

### Unisex blend sensitivity

For **EU anti-discrimination pricing** and **Solvency II** compliance studies, pass a list
of blend weights in zip mode to evaluate multiple gender-mix hypotheses in one call:

```python
from lactuca import LifeTable, TableKey

# Three blend scenarios in one call
lt40, lt50, lt60 = LifeTable("PASEM2010", "u", unisex_blend=[0.4, 0.5, 0.6])
# lt40: qx = 0.4 * qx_m + 0.6 * qx_f
# lt50: qx = 0.5 * qx_m + 0.5 * qx_f
# lt60: qx = 0.6 * qx_m + 0.4 * qx_f

# With return_dict=True: unisex_blend becomes the TableKey discriminator
tables = LifeTable("PASEM2010", "u", unisex_blend=[0.4, 0.5, 0.6], return_dict=True)
lt_mid = tables[TableKey("PASEM2010", "u", unisex_blend=0.5)]
```

In `cartesian=True` mode, `unisex_blend` must be a scalar (applied uniformly to all
combinations where `sex='u'`).  Pass a Sequence only in the default zip mode.

(bulk-portfolios)=
## Bulk portfolio calculations

For bulk work, think of the three parameters in two tiers:

- **Segment keys** (`sex`, `duration`): fixed characteristics of a policy type.  Create
  **one `LifeTable` instance per `(sex, duration)` combination** before the loop and
  never reassign them inside it.
- **Individual parameter** (`cohort`): varies per policy.  Use the `cohort` setter inside
  the loop, guarded by `if lt.cohort != c:` — the setter is idempotent (assigning the
  same value is a no-op), but the explicit guard makes the intent clear and remains
  good practice for large portfolios.

To maximize throughput, sort the portfolio by `(sex, duration, cohort)` before iterating.
This groups policies with the same cohort consecutively so the guard skips all redundant
rebuilds: N policies with K distinct cohort values cost exactly K rebuilds per segment.

The underlying `.ltk` file is read once per process — constructing or cloning a
`LifeTable` inside a loop does not re-read it.

:::{tip}
**Batch mode removes the per-policy Python loop entirely.**  Instead of calling
`lt.ax(age)` once per policy, call `äx(lt, group_ages_array, n=term)` for each
group.  This is 50–250× faster for the default `discrete_precision` mode.
See {doc}`batch_calculations` for details and performance notes.
:::

**Batch + group-then-update** — the memory-optimal pattern for large portfolios:

```python
from lactuca import LifeTable, äx

# Portfolio as parallel arrays (sort by (sex, duration, cohort) first)
portfolio = [
    {"sex": "m", "duration": 1,     "cohort": 1975, "age": 50, "term": 20},
    {"sex": "m", "duration": 1,     "cohort": 1975, "age": 48, "term": 20},
    {"sex": "f", "duration": 2,     "cohort": 1980, "age": 42, "term": 15},
    {"sex": "f", "duration": 2,     "cohort": 1984, "age": 40, "term": 15},
    {"sex": "m", "duration": "ult", "cohort": 1976, "age": 48, "term": 15},
]
portfolio.sort(key=lambda p: (p["sex"], str(p["duration"]), p["cohort"]))

# One reusable instance — O(1) tables in memory
p0 = portfolio[0]
lt = LifeTable("DAV2004R_SelUlt_1o", p0["sex"],
               cohort=p0["cohort"], duration=p0["duration"], interest_rate=0.03)

# Group by (sex, duration, cohort) and call batch ax per group
i = 0
while i < len(portfolio):
    p = portfolio[i]
    s_k, d_k, c_k, t_k = p["sex"], p["duration"], p["cohort"], p["term"]

    # Collect all policies in the same group
    j = i
    while j < len(portfolio):
        q = portfolio[j]
        if (q["sex"], q["duration"], q["cohort"], q["term"]) != (s_k, d_k, c_k, t_k):
            break
        j += 1
    group = portfolio[i:j]
    group_ages = [p["age"] for p in group]

    # Update only what changed (setters are idempotent — no rebuild on same value)
    if lt.sex != s_k:
        lt.sex = s_k
    if lt.duration != d_k:
        lt.duration = d_k
    if lt.cohort != c_k:
        lt.cohort = c_k          # recomputes qx projection; skipped if unchanged

    group_ax = äx(lt, group_ages, n=t_k)   # vectorised batch for this group

    for k, row in enumerate(group):
        row["ax"] = round(float(group_ax[k]), 4)
    i = j

print([row["ax"] for row in portfolio])
```

Sorting by `(sex, duration, cohort)` before the loop ensures each setter is called at
most once per distinct value — all policies with the same cohort are processed
consecutively, so the `lt.cohort` rebuild happens exactly K times for K distinct cohorts.



```python
from lactuca import LifeTable

ages = [35, 42, 50, 61]

lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)
results = [round(lt.ax(age, n=20, m=12), 4) for age in ages]
print(results)
```

**Generational tables** — create one instance per sex, iterate over cohorts with a guard:

```python
from lactuca import LifeTable

portfolio = [
    {"sex": "m", "cohort": 1975, "age": 50, "term": 20},
    {"sex": "f", "cohort": 1980, "age": 42, "term": 15},
    {"sex": "m", "cohort": 1975, "age": 48, "term": 20},
    {"sex": "f", "cohort": 1984, "age": 40, "term": 10},
]

tables = {
    "m": LifeTable("PER2020_Ind_2o", "m", cohort=1960, interest_rate=0.03),
    "f": LifeTable("PER2020_Ind_2o", "f", cohort=1960, interest_rate=0.03),
}

for row in sorted(portfolio, key=lambda p: (p["sex"], p["cohort"])):
    lt = tables[row["sex"]]
    if lt.cohort != row["cohort"]:       # skip if cohort unchanged
        lt.cohort = row["cohort"]
    row["ax"] = round(lt.ax(row["age"], n=row["term"], m=12), 4)

print([row["ax"] for row in portfolio])
```

**Select-ultimate tables** — one instance per `(sex, duration)` combination:

```python
from lactuca import LifeTable

portfolio = [
    {"sex": "m", "cohort": 1984, "duration": 1,     "age": 40, "term": 20},
    {"sex": "m", "cohort": 1975, "duration": 1,     "age": 50, "term": 20},
    {"sex": "f", "cohort": 1969, "duration": 2,     "age": 55, "term": 10},
    {"sex": "m", "cohort": 1976, "duration": "ult", "age": 48, "term": 15},
]

# Build one instance per (sex, duration) segment
seg_keys = {(p["sex"], p["duration"]) for p in portfolio}
tables = {
    (s, d): LifeTable(
        "DAV2004R_SelUlt_1o", s, cohort=1960, duration=d, interest_rate=0.03
    )
    for (s, d) in seg_keys
}

for row in sorted(
    portfolio,
    key=lambda p: (p["sex"], float("inf") if p["duration"] == "ult" else p["duration"], p["cohort"]),
):
    lt = tables[(row["sex"], row["duration"])]
    if lt.cohort != row["cohort"]:
        lt.cohort = row["cohort"]
    row["ax"] = round(lt.ax(row["age"], n=row["term"], m=12), 4)

print([row["ax"] for row in portfolio])
```

:::{note}
`lt.sex` and `lt.duration` each trigger a full rebuild when assigned on a real
change; both setters are idempotent (assigning the same value is a no-op).
Still, keep them out of hot loops by creating one instance per `(sex, duration)`
segment — the up-front allocation is negligible and eliminates all conditional
checks inside the loop.
:::

## Decimal precision

`lt.decimals` is a **read-only proxy** that exposes the current precision settings from
the global `Config` singleton.  Writing to `lt.decimals.xxx` raises `AttributeError`;
use `Config().decimals` to change settings:

```python
from lactuca import LifeTable, Config

lt = LifeTable("PASEM2020_Rel_1o", "m")

# Read current precision through any table instance
print(lt.decimals.qx)  # 15
print(lt.decimals.lx)  # 15
print(lt.decimals.annuities)  # 15

# Change precision via the global Config singleton
Config().decimals.annuities = 4
Config().decimals.qx = 8

# Changes are immediately reflected through all table instances
print(lt.decimals.annuities)  # 4
print(lt.decimals.qx)  # 8

# Restore factory defaults
Config().reset()
print(lt.decimals.annuities)  # 15
print(lt.decimals.qx)  # 15
```

:::{note}
Precision settings are **global**: a change via `Config().decimals` affects all table
instances in the process.  Call `Config().reset()` to restore factory defaults.
:::

The available per-quantity precision attributes:

| Attribute | Quantity |
|-----------|----------|
| `lx` | Survival function $l_x$ |
| `qx` | Decrement rates $q_x$ / $i_x$ / $o_x$ |
| `px` | Single-year survival $p_x$ |
| `tpx` | Multi-year survival ${}_tp_x$ |
| `tqx` | Multi-year mortality ${}_tq_x$ |
| `dx` | Deaths $d_x$ |
| `Lx` | $L_x$ (continuous building block) |
| `Tx` | $T_x$ |
| `ex` | Life expectancy $e_x$ |
| `Dx` | Commutation function $D_x$ |
| `Nx` | Commutation function $N_x$ |
| `Sx` | Commutation function $S_x$ |
| `Cx` | Commutation function $C_x$ |
| `Mx` | Commutation function $M_x$ |
| `Rx` | Commutation function $R_x$ |
| `annuities` | Annuity present values |
| `insurances` | Insurance present values |

See {doc}`decimals_rounding` for the full precision reference.

## Table properties

| Property | Type | Description |
|----------|------|-------------|
| `table_name` | `str` | Name of the loaded `.ltk` file |
| `sex` | `str` | Active sex: `'m'`, `'f'`, or `'u'`; settable |
| `cohort` | `int` or `None` | Active cohort year; settable (generational tables) |
| `duration` | `int`, `str`, or `None` | Active select duration; settable (select-ultimate tables) |
| `omega` | `int` | Limiting age |
| `table_type` | `str` | `"life"`, `"disability"`, or `"exit"` |
| `interest_rate` | `float` or `InterestRate` or `None` | Default interest rate; settable |
| `decimals` | `_DecimalsConfig` | Decimal precision proxy (global via `Config`) |

---

## See also

- {doc}`getting_started` — basic instantiation and first calculations
- {doc}`batch_calculations` — vectorized alternative to bulk loops: pass an age array for 50–250× speedup without iterating over policies
- {doc}`building_tables` — create custom `.ltk` files with `TableBuilder`
- {doc}`bundled_tables` — list of available bundled tables
- {doc}`mortality_improvement` — generational improvement factors and cohort projection
- {doc}`modifying_decrements` — how to adjust $q_x$ rates
- {doc}`tables_taxonomy` — Table Taxonomy: temporal and structural classification
- {doc}`decimals_rounding` — full decimal precision reference


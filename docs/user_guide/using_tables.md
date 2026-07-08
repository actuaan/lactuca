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
print(lt.ax(65, n=20))   # rounded to config.decimals.annuities (default 15)
```

Multi-scenario `InterestRate` containers can be passed as `interest_rate=` or assigned
via `lt.interest_rate = ir`.  `lt.interest_rate` holds a **reference** to the same
object — calculations without an explicit `ir=` use whichever scenario is active
**at each call**.  To freeze the scenario, pass `ir.copy()` or a simple sub-curve
from `ir.scenarios["base"]`.  See {ref}`interest-rate-scenarios-lifetable`.

The `interest_rate` property returns `InterestRate | None` — `None` until set at
construction or by assignment.

## Vectorial creation

For small families or scenario analysis, pass a sequence for `table_name`, `sex`,
`cohort` (generational tables), `duration` (select-ultimate tables), or `unisex_blend`.
When any structural parameter is a `list` or `tuple` (including length 1), when
`cartesian=True`, or when `unisex_blend` is a list, Lactuca returns a `tuple` of
independent instances (see {ref}`vectorial-return-type` below).  With every parameter
scalar, a **single** `LifeTable` is returned — no tuple.

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
print(born60.tpx(40, t=25), born70.tpx(40, t=25))  # rounded to config.decimals.tpx
```

Parameters are aligned **element-wise across all five structural parameters**
(`table_name`, `sex`, `cohort`, `duration`, and `unisex_blend` when passed as a
sequence), not as a Cartesian product.  Two table names and two sexes in zip mode produce
**2** instances — `table_name[0]` paired with `sex[0]`, and so on.  All five can vary
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
All `table_name` values in a cartesian call must share the same generational and select
structure; mixed structures are rejected up front (use zip mode instead — see
{ref}`heterogeneous-tables-zip`).

(heterogeneous-tables-zip)=
#### Heterogeneous base tables (zip mode)

Zip mode supports **different table types in one call** when each index is paired with
the `cohort` and `duration` that table requires:

```python
from lactuca import LifeTable

# Position 0: period table → cohort must be None
# Position 1: generational table → cohort required
lt_static, lt_gen = LifeTable(
    ["PASEM2020_Dec_1o", "PER2020_Ind_1o"],
    "m",
    cohort=[None, 1970],
)

# Select + non-select: duration aligned per position
lt_sel, lt_agg = LifeTable(
    ["DummyLIFE_Select", "PASEM2020_Dec_1o"],
    "m",
    duration=[1, None],
)
```

An invalid pairing (`cohort=None` on a generational table, `cohort=1970` on a period
table, wrong `duration` for a select table) raises `ValueError` when **that** instance
is constructed — align each index deliberately.  See {ref}`vectorial-constructor-errors`.

:::{important}
**Cartesian mode does not allow mixed table structures.**  If `table_name` lists
generational and period tables (or select and non-select), `cartesian=True` raises
before creating instances.  Use zip mode with aligned `cohort`/`duration` sequences
instead.
:::

(vectorial-interest-rate)=
### Default interest rate in vectorial creation

`interest_rate` can be passed directly in the constructor and will be assigned to every
vectorially created instance.  Pass a scalar to broadcast the same rate to all instances,
or a sequence of the same length to assign a distinct rate per instance:

```python
from lactuca import LifeTable, InterestRate

# Scalar broadcast — both instances get interest_rate=0.03
lt_m, lt_f = LifeTable("PASEM2020_Rel_1o", ("m", "f"), interest_rate=0.03)
print(lt_m.interest_rate)  # InterestRate: constant rate = 0.030000
print(lt_f.interest_rate)  # InterestRate: constant rate = 0.030000

# Per-instance sequence — different rates for each instance
lt_m, lt_f = LifeTable("PASEM2020_Rel_1o", ("m", "f"), interest_rate=[0.03, 0.035])
print(lt_m.interest_rate)  # InterestRate: constant rate = 0.030000
print(lt_f.interest_rate)  # InterestRate: constant rate = 0.035000

# InterestRate objects are also accepted (scalar or sequence)
ir = InterestRate(terms=[10], rates=[0.025, 0.04])
lt_m, lt_f = LifeTable("PASEM2020_Rel_1o", ("m", "f"), interest_rate=ir)
```

Passing `interest_rate=None` (the default) leaves all instances with no default rate,
which is identical to constructing them individually without the argument.

(broadcast-rules)=
### Broadcast rules

In zip mode (`cartesian=False`, the default), the instance count **N** is determined by
1-N broadcast alignment across all five structural axes (`table_name`, `sex`, `cohort`,
`duration`, `unisex_blend`).  Any parameter passed as a `list` or `tuple` — **even with
a single element** — activates the vectorial **return type** (`tuple` or `dict`); it
does not by itself increase **N** unless other axes are longer.  Scalars and length-1
sequences broadcast to match the longest aligned axis.  When two or more parameters have
length greater than 1, they must all equal **N** after alignment; any other combination
raises `ValueError`.

| Dominant sequence (length N) | Example | Instances | Return type |
|---|---|---|---|
| `table_name` | `LifeTable(["T1", "T2"], "m")` | 2 | `tuple` |
| `table_name` (length 1) | `LifeTable(["T"], "m")` | 1 | 1-`tuple` (not bare instance) |
| `sex` | `LifeTable("T", ("m", "f"))` | 2 | `tuple` |
| `cohort` | `LifeTable("T", "m", cohort=[1960, 1970])` | 2 | `tuple` |
| `duration` | `LifeTable("T", "m", duration=(1, 2))` | 2 | `tuple` |
| `unisex_blend` | `LifeTable("T", "u", unisex_blend=[0.4, 0.5])` | 2 | `tuple` |
| All matched at N | `LifeTable("T", ("m", "f"), cohort=(1960, 1963))` | 2 (zip pairs) | `tuple` |
| All scalar | `LifeTable("T", "m", cohort=1960)` | 1 | bare `LifeTable` |

`unisex_blend` as a **scalar** applies the same weight to every instance with `sex='u'`.
A **sequence** assigns one blend per instance; scalar `sex='u'` broadcasts to the list
length (you do not need `("u", "u", …)`).

`interest_rate` follows the same length rule but is applied **after** the instances are
created: a scalar (or `None`) is copied to all **N** instances; a sequence must have
length **N** to assign a distinct rate per instance (see
[Default interest rate in vectorial creation](#vectorial-interest-rate) above).

Each structural parameter can be passed as a `tuple` or a `list` — both are accepted
interchangeably.

(vectorial-return-type)=
#### Return type: instance, `tuple`, or `dict`

| Condition | `return_dict=False` (default) | `return_dict=True` |
|---|---|---|
| All parameters scalar; `cartesian=False` | Single `LifeTable` | `dict` with one `TableKey` |
| Any `list`/`tuple` on a structural axis, `cartesian=True`, or `unisex_blend` list | `tuple` of **N** instances | `dict` of **N** entries |

**Vectorial** construction activates the multi-instance path whenever `table_name`,
`sex`, `cohort`, `duration`, or `unisex_blend` is passed as a `list` or `tuple`
(including length 1), when `cartesian=True`, or when `unisex_blend` is a list.
With `return_dict=False` (default) the result is always a **`tuple`** in that path —
including when **N = 1** (e.g. `LifeTable(["PASEM2010"], "m")` returns a 1-tuple, not a
bare instance).  This is intentional: a `list`/`tuple` signals vectorial intent even
when only one table name is supplied.  Unpack with `(lt,) = ...` or `lt_m, lt_f = ...`,
index `result[0]`, or iterate.

Pass `return_dict=True` to receive
`dict[`{class}`lactuca.TableKey`, `LifeTable`]` instead: structured lookup without
positional guessing (see {ref}`tablekey-structured-keys` below).

```python
from lactuca import LifeTable, TableKey

lt = LifeTable("PASEM2010", "m")                    # scalar → bare LifeTable
lt_m, lt_f = LifeTable("PASEM2010", ("m", "f"))     # vectorial → tuple
(lt,) = LifeTable(["PASEM2010"], "m")               # 1-element list → 1-tuple

tables = LifeTable("PASEM2010", ("m", "f"), return_dict=True)
lt_m = tables[TableKey("PASEM2010", "m")]           # dict lookup
```

In **cartesian** mode (`cartesian=True`), sequences form independent axes of a full
product — there is no 1-N broadcast.  When every `sex` is `'u'`, a `unisex_blend`
sequence adds a fifth axis (`table × sex × cohort × duration × blend`).  See
{ref}`cartesian-product-creation`.

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
`duration`) — and, when every `sex` is `'u'`, (`unisex_blend` as a sequence) — in a
single constructor call.  All `table_name` values must share the same generational
structure (all period or all generational) and the same select structure; otherwise
`ValueError` is raised before any instance is created — see {ref}`heterogeneous-tables-zip`
for the zip-mode alternative.  A **scalar** `unisex_blend` is replicated uniformly to every
combo where `sex='u'`.  A **sequence** of blends adds a fifth cartesian axis (only valid
when all sex values are `'u'`).  This is appropriate for **assumption sensitivity
grids**, **pricing studies**, and **regulatory stress scenarios** where every combination
is needed regardless of whether any policy occupies it.

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
print(lt.äx(65, ir=0.03))  # rounded to config.decimals.annuities

# Unisex grid: 2 tables × 2 cohorts × 3 blend weights = 12 instances
blend_grid = LifeTable(
    ["PER2020_Ind_1o", "PER2020_Col_2o"],
    "u",
    cohort=[1960, 1970],
    unisex_blend=[0.4, 0.5, 0.6],
    cartesian=True,
    return_dict=True,
)
print(len(blend_grid))   # 12
lt_mid = blend_grid[TableKey("PER2020_Ind_1o", "u", 1960, None, 0.5)]

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

(tablekey-structured-keys)=
{class}`lactuca.TableKey` is a `NamedTuple` with five fields:

```python
from lactuca import TableKey

TableKey("PER2020_Ind_1o", "m")              # cohort=None, duration=None, unisex_blend=None
TableKey("PER2020_Ind_1o", "m", 1960)        # explicit cohort; duration=None
TableKey("PER2020_Ind_1o", "m", 1960, None)  # explicit duration
TableKey("DummyLIFE_Select", "m", 1980, "ult")  # select ultimate segment
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

:::{note}
**Lookup tips**
- Use the same `table_name` string passed to the constructor — not `instance.table_name`
  (which may differ from the repository key internally).
- `interest_rate` is **not** part of `TableKey`; it is applied per instance after
  construction.
- `TableKey("T", "m", 1960)` and `TableKey("T", "m", 1960, None, None)` are equivalent.
- To discover keys from a `return_dict=True` call: `for key in tables: print(key)`.
:::

### Unisex blend sensitivity

For **EU anti-discrimination pricing** and **Solvency II** compliance studies, pass a list
of blend weights in zip mode to evaluate multiple gender-mix hypotheses in one call.
Broadcasting rules: {ref}`broadcast-rules`.

```python
from lactuca import LifeTable, TableKey

# Three blend scenarios in one call (scalar sex="u" broadcasts to match unisex_blend)
lt40, lt50, lt60 = LifeTable("PASEM2010", "u", unisex_blend=[0.4, 0.5, 0.6])
# lt40: qx = 0.4 * qx_m + 0.6 * qx_f
# lt50: qx = 0.5 * qx_m + 0.5 * qx_f
# lt60: qx = 0.6 * qx_m + 0.4 * qx_f

# Cohort and blend zip-paired on a generational table
lt60, lt70 = LifeTable("PER2020_Col_2o", "u", cohort=[1960, 1970], unisex_blend=[0.4, 0.6])

# With return_dict=True: unisex_blend becomes the TableKey discriminator
tables = LifeTable("PASEM2010", "u", unisex_blend=[0.4, 0.5, 0.6], return_dict=True)
lt_mid = tables[TableKey("PASEM2010", "u", unisex_blend=0.5)]
```

In `cartesian=True` mode, pass `unisex_blend` as a **scalar** (uniform weight on every
`sex='u'` combo) or as a **sequence** when **all** sex values are `'u'` — the list
becomes a fifth cartesian axis.  Mixed `('m', 'u', …)` sex with a blend sequence raises
`ValueError`; use zip mode for that case.  See {ref}`cartesian-product-creation`.

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
        row["ax"] = float(group_ax[k])  # already rounded via config.decimals.annuities
    i = j

print([row["ax"] for row in portfolio])
```

Sorting by `(sex, duration, cohort)` before the loop ensures each setter is called at
most once per distinct value — all policies with the same cohort are processed
consecutively, so the `lt.cohort` rebuild happens exactly K times for K distinct cohorts.

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
    row["ax"] = lt.ax(row["age"], n=row["term"], m=12)

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
    row["ax"] = lt.ax(row["age"], n=row["term"], m=12)

print([row["ax"] for row in portfolio])
```

:::{note}
`lt.sex` and `lt.duration` each trigger a full rebuild when assigned on a real
change; both setters are idempotent (assigning the same value is a no-op).
Still, keep them out of hot loops by creating one instance per `(sex, duration)`
segment — the up-front allocation is negligible and eliminates all conditional
checks inside the loop.
:::


(deferred-construction)=
## Deferred construction: `pending`, `configure`, and `TableRegistry`

Use `pending=True` when you need to load a table's base data — improvement factors,
select segments — **before** you know the cohort, duration, or sex of the policies
in scope.  This avoids constructing a throw-away instance with dummy values just to
instantiate the class.

### `pending=True` — construct a shell

`pending=True` is valid only for **generational** or **select-ultimate** tables (period
tables have nothing to defer and raise `ValueError`):

```python
from lactuca import LifeTable

# Generational table — cohort not known yet
lt = LifeTable("PER2020_Ind_1o", "m", pending=True)

print(lt.metadata_pending)    # True
print(lt.w)                   # available — base data loaded
print(lt.generational)        # True
```

A pending table **cannot** compute `lx`, `qx`, `ax`, or any actuarial quantity — these
raise `ValueError` with an actionable message:

```
Table 'PER2020_Ind_1o' is not ready: assign 'cohort' before calculations
(lt.cohort = 1969, or lt.configure(sex="m", cohort=1969)).
```

`summary()` and `repr()` are safe to call on a pending table — they display the pending
state without computing sample values.  See {doc}`../errors_reference`
for the full error entry.

### `configure()` — assign parameters and rebuild in one step

`configure()` is the recommended way to finalize a pending table (or to reassign
multiple parameters with a **single** rebuild instead of N separate setter calls):

```python
lt.configure(sex="m", cohort=1969)   # assigns both, then rebuilds once
print(lt.metadata_pending)           # False
ax = lt.ax(65, ir=0.03)             # now available
```

`configure()` is **transactional**: if any argument fails validation the table stays in
its previous state (rollback).  It returns `self` for chaining:

```python
ax = lt.configure(cohort=1970).ax(65, ir=0.03)
```

Calling `configure()` without arguments raises `ValueError`.

For `LifeTable`, `configure()` also accepts `interest_rate=`:

```python
lt.configure(cohort=1969, interest_rate=0.03)
ax = lt.ax(65)    # uses the assigned 3 % default
```

**Unisex on a pending shell** — same rules as the constructor: if the table has no native
`q_u` / `l_u` column, pass `unisex_blend` in the same `configure()` call:

```python
lt = LifeTable("PER2020_Col_2o", pending=True)
lt.configure(sex="u", cohort=1969, unisex_blend=0.5)
ax = lt.ax(65, ir=0.03)
```

**Reconfiguring in a loop** — call `configure()` once per group to iterate over cohorts
with exactly one rebuild each time:

```python
from lactuca import LifeTable

lt = LifeTable("PER2020_Ind_1o", "m", pending=True)

for cohort in [1960, 1965, 1970, 1975]:
    result = lt.configure(cohort=cohort).ax(65, ir=0.03)
    print(f"cohort {cohort}: ax = {result:.4f}")
```

This is equivalent to four separate `LifeTable` instances but reuses the table base data
in memory (the `.ltk` file is read once; only the cohort-diagonal projection is recomputed).

:::{tip}
Sort the portfolio by `cohort` before the loop to cluster consecutive rebuilds.  Each
`configure(cohort=c)` triggers one decrement rebuild; policies in the same cohort
group can all be batched into a single functional-API call without any rebuild.
:::

### `configure_all()` — apply to a tuple of instances

After creating instances with a zip vectorial call + `pending=True`, finalize them all
with `configure_all()`:

```python
from lactuca import LifeTable, configure_all

# Two pending shells — sex known, cohort deferred
lt_m, lt_f = LifeTable("PER2020_Ind_1o", ["m", "f"], pending=True)

configure_all((lt_m, lt_f), cohort=1969)   # applies configure(cohort=1969) to each

# Optional: set a shared default interest rate for LifeTable shells
configure_all((lt_m, lt_f), cohort=1969, interest_rate=0.03)

ax_m = lt_m.ax(65, ir=0.03)
ax_f = lt_f.ax(60, ir=0.03)
```

`configure_all()` is atomic per-instance: if one fails it propagates and stops; already-
configured instances keep their new state (no global rollback).  Calling it with no
keyword arguments raises ``ValueError``.

For **select** tables created with vectorial zip + ``pending=True``, finalize duration
(and optionally sex if omitted at construction) in one pass:

```python
lt_m, lt_f = LifeTable("DummyLIFE_Select", ["m", "f"], pending=True)
configure_all((lt_m, lt_f), duration=5)
```

``duration='ult'`` is valid in ``configure()`` / ``configure_all()``; metadata fields may
be supplied in any order (for example ``configure(duration="ult")`` then
``configure(sex="m")``).

### `batch_update()` — single rebuild for multiple setters

If you prefer setter-style assignment but want only **one** rebuild, use the
`batch_update()` context manager:

```python
with lt.batch_update():
    lt.sex = "f"
    lt.cohort = 1975
    lt.duration = 1
# → exactly one decrement rebuild when the with block exits
```

An exception inside the block rolls back metadata and decrement arrays to the state before entering.
Nesting `batch_update()` raises `RuntimeError`.

### What combinations raise `ValueError`

| Combination | Reason |
|---|---|
| `pending=True` on a period (static) table | Nothing to defer |
| `pending=True` with partial scalar metadata (`sex`, `cohort`, and/or `duration`) | Values stored on the shell; table stays pending until complete |
| `pending=True` with **all** required metadata at construction | Fully configured immediately (`metadata_pending=False`) |
| `cohort=[...]` or `duration=[...]` + `pending=True` | Contradicts: enumerating and deferring the same axis |
| `cartesian=True` + `pending=True` | No clean semantics |
| `return_dict=True` + `pending=True` | `TableKey` would be stale after `configure()` |

For **dict-keyed lookup** after configuring, use `TableRegistry` (see below).

(deferred-choice)=
### When to use which tool

None of these patterns is mandatory for batch — they are ways to build a **list of
configured** `LifeTable` instances without aliasing or redundant construction.

| Situation | Recommended approach | Code example |
|---|---|---|
| Same cohort (or duration) processed **one group at a time** | `pending=True` + `configure(cohort=…)` per group | {ref}`recipe 17 <recipe-17>` ({doc}`../cookbook`) |
| Vectorial zip on sex with a **shared deferred** cohort | `pending=True` + `configure_all(..., cohort=…)` | § `configure_all()` above (this page) |
| Mixed demographics in **one batch**, keys built incrementally | `TableRegistry` + `get_or_create()` | {ref}`recipe 18 <recipe-18>` ({doc}`../cookbook`) |
| Large portfolio; all unique `(sex, cohort)` pairs **known upfront** | `return_dict=True` + `TableKey` lookup | {ref}`lookup dict <cohort-lookup-dict>` ({doc}`batch_calculations`) |
| Study grid / cartesian parameter sweep | `return_dict=True` + `cartesian=True` | {ref}`return_dict lookup <return-dict-lookup>` ({doc}`batch_calculations`) |
| Few distinct cohorts; assemble list by hand | Vectorial zip constructor | {ref}`few distinct cohorts <cohort-few-distinct>` ({doc}`batch_calculations`) |
| Very large portfolio; **memory** constrained | `groupby` + one instance + setters | {ref}`memory-optimal <cohort-memory-optimal>` ({doc}`batch_calculations`) |
| Small portfolio; metadata known per row | Direct constructor in a list comp | ``[LifeTable(..., cohort=p["cohort"]) for p in policies]`` |
| Select table; **different duration** per policy | `TableRegistry` (`duration` in cache key) | {ref}`tableregistry` below; {ref}`recipe 18 <recipe-18>` |
| Same demographic key; **different** interest rate | Pass `ir=` to the batch call | {ref}`multi-table batch <multi-table-batch-functional-api>` ({doc}`batch_calculations`) |

:::{tip}
`TableRegistry` is a convenience wrapper around the same idea as the
{ref}`lookup dict <cohort-lookup-dict>` pattern — lazy `get_or_create` with LRU
instead of building every unique key in one constructor call.
:::

API reference: {doc}`../api/table_registry`.

(tableregistry)=
### `TableRegistry` — stable instance cache for heterogeneous batch

A pending instance can represent only **one** cohort/duration at a time.  For a portfolio
where different policies require **different** cohort or duration values simultaneously,
use `TableRegistry`: it caches one configured instance per `TableKey` and never mutates
cached instances.

```python
from lactuca import LifeTable, TableRegistry

reg = TableRegistry(LifeTable)

# Cache one instance per (sex, cohort) — same key always returns the same object
lt_m60  = reg.get_or_create(None, "PER2020_Ind_1o", "m", cohort=1960)
lt_f75  = reg.get_or_create(None, "PER2020_Ind_1o", "f", cohort=1975)
lt_m60b = reg.get_or_create(None, "PER2020_Ind_1o", "m", cohort=1960)   # same cached instance
assert lt_m60 is lt_m60b
```

Build a per-policy table list for the functional batch API without aliasing risk:

```python
from lactuca import LifeTable, TableRegistry, ax

reg = TableRegistry(LifeTable)

policies = [
    {"sex": "m", "cohort": 1960, "age": 65},
    {"sex": "f", "cohort": 1975, "age": 60},
    {"sex": "m", "cohort": 1960, "age": 62},
]
tables  = [reg.get_or_create(None, "PER2020_Ind_1o", p["sex"], cohort=p["cohort"]) for p in policies]
ages    = [p["age"] for p in policies]
results = ax(tables, ages, ir=0.03)
```

`interest_rate` is **not** part of `TableKey`.  Pass `interest_rate=` (and other constructor
kwargs) only on the **first** request for a demographic key — cache hits return the existing
instance unchanged:

```python
lt = reg.get_or_create(None, "PER2020_Ind_1o", "m", cohort=1960, interest_rate=0.035)
# Same key later — same object; rate is NOT re-applied:
lt2 = reg.get_or_create(None, "PER2020_Ind_1o", "m", cohort=1960, interest_rate=0.04)
assert lt is lt2 and lt.interest_rate == 0.035
```

When policies share a demographic key but need different rates, pass `ir=` to the batch
method instead of expecting `get_or_create` to update the cached instance.

The registry is **instance-scoped** and bounded (LRU, default `maxsize=256`).  Override
`maxsize` at construction if needed.  Call `reg.clear()` to empty the cache;
`config.reset()` does **not** clear user-owned registries.  Cached instances are
**stable** — do not mutate them after retrieval — so the same object can be shared
safely across a batch call.

`TableRegistry` accepts any **concrete** decrement table class (`LifeTable`, `DisabilityTable`,
`ExitTable`) — pass the class to the constructor or to each `get_or_create()` call.
Abstract :class:`~lactuca.tables.DecrementTable` raises :exc:`TypeError`.

:::{warning}
**Aliasing with a mutated pending instance** — if you pass the same instance to multiple
entries of a `tables=` list and call `configure()` between uses, all list entries reflect
the last configured state (they all point to the same object).  Use `TableRegistry` for
heterogeneous batches to guarantee that each `TableKey` maps to a distinct stable instance.
:::

## Decimal precision

`lt.decimals` is a **read-only proxy** that exposes the current precision settings from
the global `Config` singleton.  Writing to `lt.decimals.xxx` raises `AttributeError`;
use `config.decimals` to change settings:

```python
from lactuca import LifeTable, config

lt = LifeTable("PASEM2020_Rel_1o", "m")

# Read current precision through any table instance
print(lt.decimals.qx)  # 15
print(lt.decimals.lx)  # 15
print(lt.decimals.annuities)  # 15

# Change precision via the global config singleton
config.decimals.annuities = 4
config.decimals.qx = 8

# Changes are immediately reflected through all table instances
print(lt.decimals.annuities)  # 4
print(lt.decimals.qx)  # 8

# Restore factory defaults
config.reset_to_defaults()
print(lt.decimals.annuities)  # 15
print(lt.decimals.qx)  # 15
```

:::{note}
Precision settings are **global**: a change via `config.decimals` affects all table
instances in the process.  Call `config.reset_to_defaults()` to restore factory defaults.
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
| `ex` | Complete life expectancy $\mathring{e}_x = T_x/l_x$; integer ages only; `decimals.ex` |
| `ex_curtate` | Curtate life expectancy $e_x = \sum_{k=1}^{\omega-x} {}_k p_x$; integer ages only; `decimals.ex` (terms via `decimals.tpx`) |
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
| `table_name` | `str` | Human-readable name from file metadata (may differ from the repository variable name passed to the constructor) |
| `sex` | `str` | Active sex: `'m'`, `'f'`, or `'u'`; settable |
| `cohort` | `int` or `None` | Active cohort year; settable (generational tables) |
| `duration` | `int`, `str`, or `None` | Active select duration; settable (select-ultimate tables) |
| `omega` | `int` | Limiting age |
| `table_type` | `str` | `"life"`, `"disability"`, or `"exit"` |
| `interest_rate` | `InterestRate` or `None` | Default interest rate; settable (scalar `float` inputs are normalized at construction) |
| `decimals` | read-only proxy | Exposes `config.decimals` fields (global `Config` singleton); not writable on the table instance |

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


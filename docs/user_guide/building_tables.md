# Building Custom Table Files (.ltk)

If you have your own mortality, disability or exit data that is not included in the
package, you can build and save a `.ltk` file using `TableBuilder`.  A `.ltk` file
can then be loaded by `LifeTable`, `DisabilityTable` or `ExitTable` just like any
bundled table.

The workflow is always the same:

1. Assemble your data as a `dict`, `pandas.DataFrame` or `polars.DataFrame`.
2. Construct a `TableBuilder` with the required metadata.
3. Call `save()` to write the `.ltk` file.

:::{note}
**Automatic behaviors** — `TableBuilder` handles these without requiring explicit parameters:

- **`omega` inference**: when `omega` is omitted, it is derived as
  `start_age + len(data) − 1`. Pass it explicitly only when the data vector is longer
  than the intended age range.
- **Zero-padding**: when `start_age > 0`, all columns are padded with `0.0` for ages
  0 … `start_age − 1` internally. `view_data()` hides these rows by default
  (`show_normalized=False`); pass `show_normalized=True` to expose them.
- **Validation at construction**: `validate()` is called automatically in `__init__`.
  You only need to call it manually when using `strict=False` as a non-raising
  inspection tool.
- **`grid_years` inference**: for generational tables with year-indexed MI columns
  (`mi_m_YYYY` / `mi_f_YYYY`), the calendar-year grid is inferred automatically from
  column names. Pass `grid_years` explicitly only to override the inferred value.
:::

---

## Column names and input formats

Before building any table, two things must be clear: how column names encode the table
type and sex, and what data formats `TableBuilder` accepts.

### Column naming conventions

The column name encodes three pieces of information: the decrement type (life /
disability / exit), the sex, and — for generational or select-ultimate tables — the
period or duration.  Every Lactuca table belongs to exactly one of the three types
defined in {doc}`tables_taxonomy`; the column prefix indicates which:

| Column | Type | Meaning |
|--------|------|---------|
| `qx_m` / `qx_f` | Life | Annual mortality probability, male / female |
| `qx_u` | Life | Annual mortality probability, unisex (native; see note) |
| `ix_m` / `ix_f` / `ix_u` | Disability | Disability incidence rate, male / female / unisex |
| `ox_m` / `ox_f` / `ox_u` | Exit | Exit (withdrawal) rate, male / female / unisex |
| `mi_m` / `mi_f` / `mi_u` | Any generational | Flat age-indexed improvement factor (gen-a) |
| `mi_m_YYYY` / `mi_f_YYYY` / `mi_u_YYYY` | Any generational | Year-indexed improvement factor (gen-c) |
| `qx_m_sN` / `qx_m_ult` / `qx_f_sN` / `qx_f_ult` /`qx_u_sN` / `qx_u_ult` | Select-Ultimate | Select duration N / ultimate rates |
| `mi_m_sN` / `mi_m_ult` / `mi_f_sN` / `mi_f_ult` /`mi_u_sN` / `mi_u_ult` | Select-Generational | Per-duration improvement factors (sel-gen-d) |

The `age` column must **not** be included — ages are implicit and derived from
`start_age` and `omega`.  All values must be probabilities in $[0, 1]$ (or scaled
integers when `decrement_scale_factor > 1`; see *Common construction parameters* below).

:::{note}
**Internal storage format**: `mi_m_YYYY`, `qx_m_sN / qx_m_ult`, and
`mi_m_sN / mi_m_ult` are the names used on disk and visible when calling
`view_data()`, `head()`, or `tail()`.  They are also valid as direct flat input
column names.  For generational and select-ultimate tables, a nested `dict` format
(described in the respective sections below) is often more readable than naming
every column manually.
:::

:::{note}
`qx_u` (and `ix_u`, `ox_u`) stores a pre-built unisex rate vector.  When a `.ltk` file
contains this column, `LifeTable(..., "u")` uses it directly without a blending call at
runtime.  Tables can store only `qx_m` and `qx_f` without a `qx_u` column; for the
common case of deriving a unisex blend at query time, use the `unisex_blend` parameter
of {class}`lactuca.LifeTable`, {class}`lactuca.DisabilityTable`, or
{class}`lactuca.ExitTable` (as appropriate) instead.  For the two approaches to storing a single-rate
table, see *Single-rate and sex-independent tables* below.  The unisex convention
extends to all column families: `mi_u`, `mi_u_YYYY`, `mi_u_sN / mi_u_ult`, and
`qx_u_sN / qx_u_ult` are all valid and follow the same rules as their `m` / `f`
counterparts.
:::

### Accepted input formats

**`polars.DataFrame`** is the canonical format:

```python
import polars as pl
from lactuca import TableBuilder

qx_m = [0.005 + i * 0.0003 for i in range(110)] + [1.0]  # ages 0–110; qx[omega]=1.0
qx_f = [0.004 + i * 0.0002 for i in range(110)] + [1.0]

tb = TableBuilder(
    pl.DataFrame({"qx_m": qx_m, "qx_f": qx_f}),
    table_name="MyMortality2024",
    table_type="life",
)
```

**Plain `dict`** — a flat `dict` of column → list is equivalent to a Polars DataFrame
and is convenient when building tables in pure Python:

```python
tb = TableBuilder(
    {"qx_m": qx_m, "qx_f": qx_f},
    table_name="MyMortality2024",
    table_type="life",
)
```

**`pandas.DataFrame`** — accepted for data from Excel or pandas pipelines; converted
internally via `polars.from_pandas()`:

```python
import pandas as pd

df_pd = pd.DataFrame({"qx_m": qx_m, "qx_f": qx_f})
tb = TableBuilder(df_pd, table_name="MyMortality2024", table_type="life")
```

:::{note}
If you already have your data as a Polars DataFrame, pass it directly to avoid the
`pandas` → `polars` conversion cost.
:::

---

## Common construction parameters

These parameters apply to all table types and are explained here once.

### Age range: `start_age` and `omega`

By default, `start_age=0` and `omega` is inferred from the data length.  If your data
covers only a subset of ages — for instance, a working-population table for ages 18–65
— set both explicitly:

```python
tb = TableBuilder(
    df_18_to_65,
    table_name="WorkforceTable",
    table_type="exit",
    start_age=18,
    omega=65,
)
```

Lactuca pads ages below `start_age` internally with zero rates so the internal arrays
always start at age 0.  These padding rows are hidden by `view_data()` and `head()` by
default; pass `show_normalized=True` to expose them.

### Scaled rates: `decrement_scale_factor` and `mi_scale_factor`

Many published tables express rates in per-mille (×1 000) or per-ten-thousand (×10 000)
to avoid small decimals in print.  Pass the factor; the engine divides by it when
loading the file:

```python
# qx expressed as e.g. 2.5 meaning 0.0025
tb = TableBuilder(
    pl.DataFrame({
        "qx_m": [2.5, 3.1, 4.0],   # × 1 000 rates
        "qx_f": [1.8, 2.3, 3.2],
    }),
    table_name="TablePerMille",
    table_type="life",
    start_age=108,
    decrement_scale_factor=1000,
    # mi_scale_factor=1000,  # use this as well if mi_* columns are also per-mille
)
```

Valid values for both factors: any power of 10 (1, 10, 100, 1000, …).

### Saving and file paths

```python
tb.save(
    file_name="MyMortality2024.ltk",  # defaults to table_name + ".ltk" when None
    path="actuarial_tables",           # directory; None uses Config.tables_path
    overwrite=False,                   # True to replace an existing file
)
```

:::{note}
**`save()` — paths, overwrites, and atomicity**

- **Path boundary**: when `path` is `None`, the target is restricted to
  `Config().tables_path`; writes outside it raise `ValueError`. An explicit `path`
  bypasses this check and writes directly to the given directory.
- **Idempotent overwrite**: when `overwrite=False` and an existing file has **identical
  content**, `save()` emits a `UserWarning` and returns without raising — re-running
  the same build script is safe.
- **Atomic write**: data is first written to a temporary file in the target directory
  and then moved into place with `os.replace()`, making writes safe against power
  failures mid-write.
:::

---

## Single-rate and sex-independent tables

Some published tables provide a single rate vector with no sex breakdown.  This pattern
applies to any table type (life, disability, exit) and comes in two variants.

### Option A — native unisex column (`qx_u`)

Store the rates under `qx_u` (or `ix_u` / `ox_u`).  Instantiate with `sex='u'`
**without** `unisex_blend`:

```python
import polars as pl
from lactuca import TableBuilder, LifeTable

qx = [0.004 + i * 0.0002 for i in range(110)] + [1.0]  # qx[omega]=1.0 required

tb = TableBuilder(
    pl.DataFrame({"qx_u": qx}),
    table_name="MyUnisexTable",
    table_type="life",
    description="Single-rate unisex mortality table.",
)
tb.save(path="actuarial_tables")

lt = LifeTable("MyUnisexTable", "u", interest_rate=0.03)
print(lt.äx(65))
```

If you supply **only** `qx_u` (no `qx_m` / `qx_f`), then `sex='m'` and `sex='f'` are
not available.  You can also combine all three — `qx_m`, `qx_f`, **and** `qx_u` — to
produce a table with `valid_sexes = ['f', 'm', 'u']` where each sex uses its dedicated
column and no blending is ever needed.

### Option B — mirrored sex column (`sex_independent=True`)

Store the rate under a conventional sex column (e.g. `qx_m`) and set
`sex_independent=True`.  The engine mirrors the column to the other sex at load time,
so both `"m"` and `"f"` instantiation work:

```python
tb = TableBuilder(
    pl.DataFrame({"qx_m": qx}),
    table_name="MySexIndepTable",
    table_type="life",
    sex_independent=True,
    description="Single-rate table, mirrored to both sexes.",
)
tb.save(path="actuarial_tables")
```

After saving, load it with either sex — both access the same rates:

```python
lt = LifeTable("MySexIndepTable", "m", interest_rate=0.03)
# LifeTable("MySexIndepTable", "f", ...) produces identical results
print(lt.äx(65))
```

This also applies to disability tables — for example, the IASS-90 inception table
publishes a single rate for both sexes; replace `qx_m` with `ix_m` and set
`table_type="disability"`.

After loading, `valid_sexes` is `['f', 'm']` — `"u"` is **not** added automatically.
Using `sex='u'` still requires `unisex_blend`, but because `qx_m == qx_f` identically,
the blend weight has no effect on the result.

| | Option A (`qx_u`) | Option B (`sex_independent`) |
|---|---|---|
| Available sexes | `"u"` (plus `"m"`, `"f"` if those columns are also provided) | `"m"`, `"f"` (and `"u"` with `unisex_blend`) |
| `unisex_blend` required for `sex='u'`? | No | Yes (though numerically irrelevant) |
| Recommended when… | Source is already a unified rate | You need sex-specific API access |

:::{note}
`sex_independent` is a **metadata flag only**.  It does not create a `qx_u` / `ix_u`
column.  The valid sexes after loading are `['f', 'm']`, not `['f', 'm', 'u']`.  If
you need `sex='u'` without a blending call, use a native `qx_u` column instead.
:::

---

## Aggregate – Static tables

The simplest category: one row per age, sex-specific rate columns, no improvement
factors.  Corresponds to *Aggregate – Static* in {doc}`tables_taxonomy`.

### Life table

```python
import polars as pl
from lactuca import TableBuilder, LifeTable

qx_m = [0.005 + i * 0.0003 for i in range(110)] + [1.0]  # ages 0–110; qx[omega]=1.0
qx_f = [0.004 + i * 0.0002 for i in range(110)] + [1.0]

tb = TableBuilder(
    pl.DataFrame({"qx_m": qx_m, "qx_f": qx_f}),
    table_name="MyMortality2024",
    table_type="life",
    description="Illustrative static mortality table, both sexes, ages 0–110.",
)

tb.validate()       # raises ValueError on any structural or range problem
print(tb.summary()) # human-readable metadata summary

tb.save(path="actuarial_tables")
```

After saving, load it like any other table:

```python
lt = LifeTable("MyMortality2024", "m", interest_rate=0.03)
print(lt.äx(65))
```

### Disability table

Use `table_type="disability"` with the `ix_` column prefix:

```python
import polars as pl
from lactuca import TableBuilder

tb_dis = TableBuilder(
    pl.DataFrame({
        "ix_m": [0.002, 0.003, 0.004],
        "ix_f": [0.001, 0.002, 0.003],
    }),
    table_name="CustomDisability",
    table_type="disability",
    start_age=63,
    omega=65,
    description="Custom disability incidence table, ages 63–65.",
)
tb_dis.save(path="actuarial_tables")
```

To build a unisex disability table, supply `ix_u` in place of `ix_m` / `ix_f` and
instantiate with `DisabilityTable(..., "u")`.  See *Single-rate and sex-independent
tables* above.

### Exit table

Use `table_type="exit"` with the `ox_` prefix:

```python
import polars as pl
from lactuca import TableBuilder

tb_exit = TableBuilder(
    pl.DataFrame({
        "ox_m": [0.05, 0.04, 0.03],
        "ox_f": [0.04, 0.03, 0.02],
    }),
    table_name="CustomExit",
    table_type="exit",
    start_age=63,
    omega=65,
)
tb_exit.save(path="actuarial_tables")
```

To build a unisex exit table, supply `ox_u` in place of `ox_m` / `ox_f` and
instantiate with `ExitTable(..., "u")`.  See *Single-rate and sex-independent
tables* above.

---

## Aggregate – Generational tables

A generational table extends any static table with *mortality improvement factors* that
project base rates forward in time for each cohort.  See {doc}`tables_taxonomy` and
{doc}`mortality_improvement` for the projection formulas and bundled examples.

Three parameters are always required in addition to the base columns:

- `generational=True`
- `base_year` — reference year $t_0$ for the projection
- `generational_formula_type` — one of:

| Value | Formula | Typical use |
|-------|---------|-------------|
| `"exponential_improvement"` | $q_x(t) = q_x^0 \cdot e^{-\lambda_x(t-t_0)}$ | PER 2020, DAV 2004 R |
| `"linear_improvement"` | $q_x(t) = q_x^0 - mi_x \cdot (t-t_0)$ | Research / custom |
| `"discrete_improvement"` | $q_x(c) = q_x^0 \cdot (1 - AA_x)^{c+x-t_0}$ | GAM 94 / SOA Scale AA |
| `"projected_improvement"` | $q_{x,c} = q_{x,t_0} \cdot \prod_{t=t_0+1}^{c+x}(1-AA_{x,t})$ | Chilean CMF 2020 |

### Flat improvement factors per age (agg–gen-a)

One scalar improvement factor per age (`mi_m`, `mi_f`), uniform across all projection
years.  This is the simplest generational variant:

```python
import polars as pl
from lactuca import TableBuilder, LifeTable

qx_m = [0.003 + i * 0.0002 for i in range(110)] + [1.0]   # ages 0–110; qx[omega]=1.0
qx_f = [0.002 + i * 0.00015 for i in range(110)] + [1.0]
mi_m = [0.015] * len(qx_m)   # 1.5 % annual improvement, all ages
mi_f = [0.012] * len(qx_f)

tb = TableBuilder(
    pl.DataFrame({"qx_m": qx_m, "qx_f": qx_f, "mi_m": mi_m, "mi_f": mi_f}),
    table_name="GenMortality2024",
    table_type="life",
    generational=True,
    base_year=2024,
    generational_formula_type="exponential_improvement",
    description="Illustrative generational life table, base year 2024.",
)
tb.save(path="actuarial_tables")
```

Load with a cohort year:

```python
lt = LifeTable("GenMortality2024", "m", cohort=1970, interest_rate=0.03)
print(round(lt.äx(65), 4))
```

For a unisex generational table, pass `mi_u` in place of `mi_m` / `mi_f`.

### Discrete annual scale factor (agg–gen-b)

When improvement rates are published as discrete annual scale factors — as in the
SOA *Scale AA* used with GAM 94 — pass
`generational_formula_type="discrete_improvement"`.  The column structure is identical
to *agg–gen-a* (flat `mi_m` / `mi_f` per age), but the formula accumulates the factor
as a discrete annual product:

$$q_x(c) = q_x^0 \cdot (1 - mi_x)^{c + x - t_0}$$

where $c$ is the birth cohort, $x$ is the attained age, and $t_0$ is `base_year`.
The `mi_m` / `mi_f` column holds the improvement factor $mi_x$ for each age
(e.g., `0.018` means a 1.8 % annual improvement at that age).

```python
import polars as pl
from lactuca import TableBuilder

n = 120                                        # ages 1–120
qx_m = [min(0.001 + i * 0.001, 1.0) for i in range(119)] + [1.0]  # qx[omega]=1.0 required
qx_f = [min(0.0007 + i * 0.00075, 1.0) for i in range(119)] + [1.0]
# Scale AA improvement rates per age (steadily declining with age)
mi_m = [max(0.018 - i * 0.00005, 0.0) for i in range(n)]
mi_f = [max(0.015 - i * 0.00004, 0.0) for i in range(n)]

tb = TableBuilder(
    pl.DataFrame({"qx_m": qx_m, "qx_f": qx_f, "mi_m": mi_m, "mi_f": mi_f}),
    table_name="DiscreteGenTable",
    table_type="life",
    generational=True,
    base_year=2000,
    generational_formula_type="discrete_improvement",
    start_age=1,
    description="Illustrative discrete-scale generational table (Scale AA style), base year 2000.",
)
tb.save(path="actuarial_tables")
```

Load with a cohort year:

```python
lt = LifeTable("DiscreteGenTable", "m", cohort=1960, interest_rate=0.03)
print(round(lt.äx(65), 4))
```

For a unisex generational table, pass `mi_u` in place of `mi_m` / `mi_f`.

### Year-indexed MI grid (agg–gen-c)

When improvement factors are published as an annual matrix — one factor per age per
calendar year — use `generational_formula_type="projected_improvement"`.  `grid_years`
is **inferred automatically** from the column names.

There are two equivalent ways to supply the MI data:

**Flat columns** (`mi_m_YYYY` / `mi_f_YYYY`):

```python
import polars as pl
from lactuca import TableBuilder

years = list(range(2010, 2036))   # 26-year MI grid
qx_m = [0.004 + i * 0.0003 for i in range(110)] + [1.0]   # ages 0–110; qx[omega]=1.0
qx_f = [0.003 + i * 0.0002 for i in range(110)] + [1.0]

mi_m_cols = {f"mi_m_{y}": [0.015 - 0.0001 * (y - 2010)] * 111 for y in years}
mi_f_cols = {f"mi_f_{y}": [0.012 - 0.0001 * (y - 2010)] * 111 for y in years}

tb = TableBuilder(
    pl.DataFrame({"qx_m": qx_m, "qx_f": qx_f, **mi_m_cols, **mi_f_cols}),
    table_name="GenC_2010_2035",
    table_type="life",
    generational=True,
    base_year=2009,              # must be strictly less than the first grid year (2010)
    generational_formula_type="projected_improvement",
    description="Year-indexed MI table, gen-c, years 2010–2035.",
)
tb.save(path="actuarial_tables")
```

**Nested `dict`** (more readable — each `mi_*` value is a `{year: [rates]}` dict):

```python
tb = TableBuilder(
    {
        "qx_m": qx_m,
        "qx_f": qx_f,
        "mi_m": {y: [0.015 - 0.0001 * (y - 2010)] * 111 for y in years},
        "mi_f": {y: [0.012 - 0.0001 * (y - 2010)] * 111 for y in years},
    },
    table_name="GenC_2010_2035",
    table_type="life",
    generational=True,
    base_year=2009,              # must be strictly less than the first grid year (2010)
    generational_formula_type="projected_improvement",
)
```

Both forms produce the same internal `mi_m_YYYY` / `mi_f_YYYY` column layout.  See
{doc}`tables_taxonomy` (section *Aggregate – Generational, sub-type c*) for the
projection formula.

For a unisex generational table, use `mi_u_YYYY` columns (or `"mi_u": {year: [rates]}`
in nested dict form) in place of the `m` / `f` variants.

---

## Select-Ultimate – Static tables

A select-ultimate table stores separate rate columns for each select duration plus an
ultimate column.  Pass `select=True` and `select_period=N`, where **`N` is the count
of numbered select-duration columns per sex** (not the maximum duration number).

Lactuca **always requires an explicit ultimate column**, named `_ult` (or key `"ult"`
in the nested-dict format).  The ultimate column holds the rates that apply for all
durations ≥ `start_duration + select_period`.  This is a mandatory column for all
select-ultimate tables, regardless of how the source data is labeled.

For most tables, durations are numbered starting at 1 (columns `_s1` … `_sN` plus
`_ult`).  For CMI / UK-style tables such as AM92 and AF92, durations start at 0:
use integer keys `0`, `1`, …, `N-1` in the nested-dict format (or flat column names
`_s0` … `_s(N-1)`); the starting duration is auto-detected from the minimum key
present.

:::{note}
CMI (UK) published tables typically list the aggregate (ultimate) rates as a third
numbered column alongside the select columns — for example, AM92 shows durations 0, 1,
and the aggregate labelled by calendar age or "duration 2+".  **Do not pass that
aggregate column as an integer key** (e.g. `2`) in the nested dict: Lactuca
would treat it as a third select duration, which is wrong.  Map it to the string
key `"ult"` (or name it `_ult` in the flat format) to indicate it is the permanent
ultimate rate.  Example — AM92 with a 2-year select period: `select_period=2`, keys
`{0: [...], 1: [...], "ult": [...]}` → columns `qx_m_s0`, `qx_m_s1`, `qx_m_ult`.
:::

Corresponds to *Select-Ultimate – Static* in {doc}`tables_taxonomy`.

There are two equivalent ways to supply the data:

**Flat columns** — name every column explicitly:

```python
import polars as pl
from lactuca import TableBuilder

tb = TableBuilder(
    pl.DataFrame({
        "qx_m_s1":  [0.0003, 0.0004, 0.0006],  # duration 1 (first year after selection)
        "qx_m_s2":  [0.0005, 0.0007, 0.0009],  # duration 2
        "qx_m_ult": [0.0008, 0.0010, 0.0013],  # ultimate (duration ≥ 3)
        "qx_f_s1":  [0.0002, 0.0003, 0.0004],
        "qx_f_s2":  [0.0003, 0.0005, 0.0007],
        "qx_f_ult": [0.0006, 0.0008, 0.0011],
    }),
    table_name="MySelectTable",
    table_type="life",
    select=True,
    select_period=2,
    start_age=108,
    description="Select-ultimate table with 2-year select period.",
)
tb.save(path="actuarial_tables")
```

**Nested `dict`** — each sex key maps to a `{duration: [rates]}` dict; integer keys
become `_sN` suffixes and `"ult"` becomes `_ult`:

```python
tb = TableBuilder(
    {
        "qx_m": {
            1:     [0.0003, 0.0004, 0.0006],
            2:     [0.0005, 0.0007, 0.0009],
            "ult": [0.0008, 0.0010, 0.0013],
        },
        "qx_f": {
            1:     [0.0002, 0.0003, 0.0004],
            2:     [0.0003, 0.0005, 0.0007],
            "ult": [0.0006, 0.0008, 0.0011],
        },
    },
    table_name="MySelectTable",
    table_type="life",
    select=True,
    select_period=2,
    start_age=108,
)
```

Both forms produce identical `qx_m_s1`, `qx_m_s2`, `qx_m_ult`, … columns.

For unisex rates, use `qx_u_sN` / `qx_u_ult` columns (or `"qx_u": {duration: [...], "ult": [...]}`
in nested dict form) in place of the sex-specific equivalents.

---

## Select-Ultimate – Generational tables

A select-ultimate table can also carry mortality improvement factors.  Pass both
`select=True` and `generational=True` together with the standard improvement
parameters.  This is the pattern used by the *DAV 2004 R Selektionstafeln* and
similar tables that distinguish select-period rates from ultimate rates *and* project
both forward for each cohort.  Corresponds to *Select-Ultimate – Generational,
sub-type a* in {doc}`tables_taxonomy`.

### Uniform improvement factors per age (sel–gen-a)

**Flat columns** — one `mi_m` / `mi_f` vector applies to all durations (select and
ultimate) uniformly:

```python
import polars as pl
from lactuca import TableBuilder

n = 111   # ages 0–110
qx_m_s1  = [0.0003 + i * 0.00015 for i in range(n)]
qx_m_s2  = [0.0005 + i * 0.00018 for i in range(n)]
qx_m_ult = [0.0008 + i * 0.00022 for i in range(n)]
qx_f_s1  = [0.0002 + i * 0.00010 for i in range(n)]
qx_f_s2  = [0.0003 + i * 0.00013 for i in range(n)]
qx_f_ult = [0.0006 + i * 0.00017 for i in range(n)]
mi_m     = [0.015] * n
mi_f     = [0.012] * n

tb = TableBuilder(
    pl.DataFrame({
        "qx_m_s1": qx_m_s1, "qx_m_s2": qx_m_s2, "qx_m_ult": qx_m_ult,
        "qx_f_s1": qx_f_s1, "qx_f_s2": qx_f_s2, "qx_f_ult": qx_f_ult,
        "mi_m": mi_m, "mi_f": mi_f,
    }),
    table_name="SelGenTable",
    table_type="life",
    select=True,
    select_period=2,
    generational=True,
    base_year=1999,
    generational_formula_type="exponential_improvement",
    description="Illustrative select-ultimate generational table, select period 2.",
)
tb.save(path="actuarial_tables")
```

**Nested `dict`** — identical result; each sex key maps to `{duration: [rates]}`
and `mi_*` are flat arrays:

```python
tb = TableBuilder(
    {
        "qx_m": {
            1:     qx_m_s1,
            2:     qx_m_s2,
            "ult": qx_m_ult,
        },
        "qx_f": {
            1:     qx_f_s1,
            2:     qx_f_s2,
            "ult": qx_f_ult,
        },
        "mi_m": mi_m,
        "mi_f": mi_f,
    },
    table_name="SelGenTable",
    table_type="life",
    select=True,
    select_period=2,
    generational=True,
    base_year=1999,
    generational_formula_type="exponential_improvement",
)
```

Both forms produce identical `qx_m_s1`, `qx_m_s2`, `qx_m_ult`, `mi_m`, … columns.
Load with both a `duration` (select period elapsed) and a `cohort`:

```python
from lactuca import LifeTable

lt = LifeTable("SelGenTable", "m", duration=1, cohort=1980, interest_rate=0.03)
print(round(lt.äx(65), 4))
```

For a unisex select-generational table, use `qx_u_s*` / `qx_u_ult` and `mi_u` in
place of the `m` / `f` variants.

### Year-indexed MI grid, uniform across durations (sel–gen-b)

The year-indexed variant uses `mi_m_YYYY` / `mi_f_YYYY` columns (one factor per age
per calendar year) and applies them **uniformly to all durations**.  Any of the three
time-constant formulas (`exponential_improvement`, `linear_improvement`,
`discrete_improvement`) can be used; the projection diagonal for each insured is
$t = \text{cohort} + x$.  This is the structure of *DAV 2004 R SelUlt 2. Ordnung*
(`DAV2004R_SelUlt_2o`).

```python
import polars as pl
from lactuca import TableBuilder

n = 111
years = list(range(2000, 2040))   # 40-year MI grid

qx_m_s1  = [0.0003 + i * 0.00015 for i in range(n)]
qx_m_s2  = [0.0005 + i * 0.00018 for i in range(n)]
qx_m_ult = [0.0008 + i * 0.00022 for i in range(n)]
qx_f_s1  = [0.0002 + i * 0.00010 for i in range(n)]
qx_f_s2  = [0.0003 + i * 0.00013 for i in range(n)]
qx_f_ult = [0.0006 + i * 0.00017 for i in range(n)]

mi_m_cols = {f"mi_m_{y}": [0.015 - 0.0001 * (y - 2000)] * n for y in years}
mi_f_cols = {f"mi_f_{y}": [0.012 - 0.0001 * (y - 2000)] * n for y in years}

tb = TableBuilder(
    pl.DataFrame({
        "qx_m_s1": qx_m_s1, "qx_m_s2": qx_m_s2, "qx_m_ult": qx_m_ult,
        "qx_f_s1": qx_f_s1, "qx_f_s2": qx_f_s2, "qx_f_ult": qx_f_ult,
        **mi_m_cols, **mi_f_cols,
    }),
    table_name="SelGenB_Table",
    table_type="life",
    select=True,
    select_period=2,
    generational=True,
    base_year=1999,              # strictly less than the first grid year (2000)
    generational_formula_type="exponential_improvement",
    description="Illustrative sel-gen-b table: year-indexed MI, uniform across durations.",
)
tb.save(path="actuarial_tables")
```

**Nested `dict`** — nest `qx` columns by duration and `mi` columns by year:

```python
tb = TableBuilder(
    {
        "qx_m": {
            1:     qx_m_s1,
            2:     qx_m_s2,
            "ult": qx_m_ult,
        },
        "qx_f": {
            1:     qx_f_s1,
            2:     qx_f_s2,
            "ult": qx_f_ult,
        },
        "mi_m": {y: [0.015 - 0.0001 * (y - 2000)] * n for y in years},
        "mi_f": {y: [0.012 - 0.0001 * (y - 2000)] * n for y in years},
    },
    table_name="SelGenB_Table",
    table_type="life",
    select=True,
    select_period=2,
    generational=True,
    base_year=1999,
    generational_formula_type="exponential_improvement",
)
tb.save(path="actuarial_tables")
```

Both forms produce identical `qx_m_s1`, `qx_m_s2`, `qx_m_ult`, `mi_m_2000`, … columns.

For a unisex variant, use `qx_u_s*` / `qx_u_ult` and `mi_u_YYYY` / `"mi_u": {year: [rates]}`
in place of the `m` / `f` variants.

### Year-indexed MI grid, cumulative product (sel–gen-c)

Combines select-ultimate rates with year-indexed improvement $mi_{x,t}$ applied via
the **cumulative product** formula (`projected_improvement`).  The projection follows
each insured's diagonal $t = \text{cohort} + x + d$ (where $d$ is the elapsed duration
since underwriting):

$$
q_{[x]+d,\,\text{cohort}}
  = q_{[x]+d,\,t_0}
    \cdot \prod_{t=t_0+1}^{\text{cohort}+x+d} \bigl(1 - mi_{x+d,\,t}\bigr)
$$

The column structure is identical to *sel–gen-b*, but
`generational_formula_type="projected_improvement"`.  This is the most technically
rigorous structure for life-insurance portfolios with both underwriting selection and
year-by-year improvement projections (VBT, IRS mortality improvement scales).

```python
import polars as pl
from lactuca import TableBuilder

n = 111
years = list(range(2010, 2036))   # 26-year MI grid

qx_m_s1  = [0.0003 + i * 0.00015 for i in range(n)]
qx_m_ult = [0.0008 + i * 0.00022 for i in range(n)]
qx_f_s1  = [0.0002 + i * 0.00010 for i in range(n)]
qx_f_ult = [0.0006 + i * 0.00017 for i in range(n)]

mi_m_cols = {f"mi_m_{y}": [0.015 - 0.0001 * (y - 2010)] * n for y in years}
mi_f_cols = {f"mi_f_{y}": [0.012 - 0.0001 * (y - 2010)] * n for y in years}

tb = TableBuilder(
    pl.DataFrame({
        "qx_m_s1": qx_m_s1, "qx_m_ult": qx_m_ult,
        "qx_f_s1": qx_f_s1, "qx_f_ult": qx_f_ult,
        **mi_m_cols, **mi_f_cols,
    }),
    table_name="SelGenC_Table",
    table_type="life",
    select=True,
    select_period=1,
    generational=True,
    base_year=2009,              # strictly less than the first grid year (2010)
    generational_formula_type="projected_improvement",
    description="Illustrative sel-gen-c table: projected_improvement, select period 1.",
)
tb.save(path="actuarial_tables")
```

**Nested `dict`** — nest `qx` columns by duration and `mi` columns by year:

```python
tb = TableBuilder(
    {
        "qx_m": {
            1:     qx_m_s1,
            "ult": qx_m_ult,
        },
        "qx_f": {
            1:     qx_f_s1,
            "ult": qx_f_ult,
        },
        "mi_m": {y: [0.015 - 0.0001 * (y - 2010)] * n for y in years},
        "mi_f": {y: [0.012 - 0.0001 * (y - 2010)] * n for y in years},
    },
    table_name="SelGenC_Table",
    table_type="life",
    select=True,
    select_period=1,
    generational=True,
    base_year=2009,
    generational_formula_type="projected_improvement",
)
tb.save(path="actuarial_tables")
```

Both forms produce identical `qx_m_s1`, `qx_m_ult`, `mi_m_2010`, … columns.

For a unisex variant, use `qx_u_s*` / `qx_u_ult` and `mi_u_YYYY` / `"mi_u": {year: [rates]}`
in place of the `m` / `f` variants.

### Per-duration improvement vectors (sel–gen-d)

The most granular variant: each select duration (and the ultimate) carries its own
improvement vector (`mi_m_s1`, `mi_m_s2`, …, `mi_m_ult` and female equivalents).
The formula is applied **independently per duration** with diagonal $t = \text{cohort} + x$:

| Formula | Projected rate $q_{[x]+d,t}$ |
|---|---|
| `exponential_improvement` | $q_{[x]+d,t_0} \cdot e^{-\lambda_x^{[d]}(t-t_0)}$ |
| `linear_improvement` | $q_{[x]+d,t_0} - mi_x^{[d]} \cdot (t-t_0)$ |
| `discrete_improvement` | $q_{[x]+d,t_0} \cdot (1 - AA_x^{[d]})^{t-t_0}$ |

Column names follow the pattern `mi_{sex}_s{k}` / `mi_{sex}_ult`:

```python
import polars as pl
from lactuca import TableBuilder

n = 111
qx_m_s1  = [0.0003 + i * 0.00015 for i in range(n)]
qx_m_s2  = [0.0005 + i * 0.00018 for i in range(n)]
qx_m_ult = [0.0008 + i * 0.00022 for i in range(n)]
qx_f_s1  = [0.0002 + i * 0.00010 for i in range(n)]
qx_f_s2  = [0.0003 + i * 0.00013 for i in range(n)]
qx_f_ult = [0.0006 + i * 0.00017 for i in range(n)]

# Each duration and ultimate has its own improvement rates
mi_m_s1  = [0.020] * n    # higher improvement in select period 1 (healthier lives)
mi_m_s2  = [0.018] * n
mi_m_ult = [0.015] * n    # lower improvement in ultimate
mi_f_s1  = [0.017] * n
mi_f_s2  = [0.015] * n
mi_f_ult = [0.012] * n

tb = TableBuilder(
    pl.DataFrame({
        "qx_m_s1": qx_m_s1, "qx_m_s2": qx_m_s2, "qx_m_ult": qx_m_ult,
        "qx_f_s1": qx_f_s1, "qx_f_s2": qx_f_s2, "qx_f_ult": qx_f_ult,
        "mi_m_s1": mi_m_s1, "mi_m_s2": mi_m_s2, "mi_m_ult": mi_m_ult,
        "mi_f_s1": mi_f_s1, "mi_f_s2": mi_f_s2, "mi_f_ult": mi_f_ult,
    }),
    table_name="SelGenD_Table",
    table_type="life",
    select=True,
    select_period=2,
    generational=True,
    base_year=2020,
    generational_formula_type="exponential_improvement",
    description="Illustrative sel-gen-d table: per-duration improvement vectors.",
)
tb.save(path="actuarial_tables")
```

**Nested `dict`** — nest both `qx` and `mi` columns by duration; integer keys become
`_sN` suffixes and `"ult"` becomes `_ult`:

```python
tb = TableBuilder(
    {
        "qx_m": {
            1:     qx_m_s1,
            2:     qx_m_s2,
            "ult": qx_m_ult,
        },
        "qx_f": {
            1:     qx_f_s1,
            2:     qx_f_s2,
            "ult": qx_f_ult,
        },
        "mi_m": {
            1:     mi_m_s1,
            2:     mi_m_s2,
            "ult": mi_m_ult,
        },
        "mi_f": {
            1:     mi_f_s1,
            2:     mi_f_s2,
            "ult": mi_f_ult,
        },
    },
    table_name="SelGenD_Table",
    table_type="life",
    select=True,
    select_period=2,
    generational=True,
    base_year=2020,
    generational_formula_type="exponential_improvement",
)
tb.save(path="actuarial_tables")
```

Both forms produce identical `qx_m_s1`, `qx_m_s2`, `qx_m_ult`, `mi_m_s1`, … columns.

---

## Inspecting a `TableBuilder`

Three methods let you explore the contents of a `TableBuilder` instance at any stage —
before or after saving.

`summary()` returns a multi-line string with the key metadata:

```python
tb = TableBuilder.from_file("MyMortality2024.ltk", path="actuarial_tables")
print(tb.summary())
# Life Table: 'MyMortality2024'
# Valid sexes: m, f
# Age range: 0–110 (start_age=0, omega=110)
# Decrement scale factor applied: 1
# Generational: False
```

`view_data()` returns a `polars.DataFrame` with an optional `age` column.  By default
it shows only meaningful rows (from `start_age` to `omega`), hiding internal zero-padding
for ages below `start_age`:

```python
df     = tb.view_data()                          # age + data columns, start_age → omega
df_all = tb.view_data(show_normalized=True)      # includes zero-padded rows
df_raw = tb.view_data(include_age=False)         # data columns only, no age column
```

`head(n)` and `tail(n)` return the first or last *n* rows of meaningful data:

```python
print(tb.head(5))   # first 5 rows (ages start_age … start_age + 4)
print(tb.tail(5))   # last 5 rows  (ages omega − 4 … omega)
```

`str(tb)` (or `repr(tb)`) returns a compact single-line summary — useful for logging
and quick diagnostics:

```python
>>> print(tb)
<Life Table: 'MyMortality2024' | Valid sexes: m, f | Age range: 0–110 (start_age=0, omega=110) | Decrement scale factor applied: 1 | Generational: False>
```

### Validation scope and non-strict mode

`validate()` checks the following (raising `ValueError` by default when `strict=True`):

- No null values in any column
- Column names follow the expected naming conventions for the table type (`qx_` / `ix_` / `ox_` prefixes, valid sex suffixes, valid duration / year suffixes)
- All decrement values are in $[0, 1]$
- Data length is consistent with `omega − start_age + 1`
- Select-ultimate tables have exactly `select_period` numbered duration columns plus `_ult` per sex
- Generational tables: `generational_formula_type` is one of the four recognised values, `base_year` is present and > 1900; for `projected_improvement` tables `base_year` must be strictly less than the minimum `grid_years` entry
- Cohort extrapolation sanity: for time-constant formulas (exponential / linear / discrete improvement), six representative cohorts are projected and the resulting probabilities are verified to stay in $[0, 1]$

Pass `strict=False` to get a `bool` result and a `UserWarning` instead of an
exception — useful for batch checks or interactive exploration:

```python
ok = tb.validate(strict=False)
if not ok:
    print("Table has structural issues — check warnings above.")
```

---

## Loading, round-tripping and cloning

### Loading a saved table

```python
from lactuca import TableBuilder, read_table

# Load back as a TableBuilder (strict integrity check; raises on hash mismatch)
tb2 = TableBuilder.from_file("MyMortality2024.ltk", path="actuarial_tables", strict=True)
print(tb2.summary())

# Or load just the DataFrame and metadata dict (lightweight, no TableBuilder overhead).
# validate defaults to False (best-effort / recovery mode).
# Pass validate=True to enforce strict integrity checking:
df, meta = read_table("MyMortality2024.ltk", path="actuarial_tables", validate=True)
print(meta["table_name"])   # 'MyMortality2024'
print(meta["omega"])        # '110'  (string — all metadata values are strings)
print(df.head())

# Read with recovery mode (useful for inspecting potentially malformed files):
df_rec, meta_rec = read_table(
    "MyMortality2024.ltk", path="actuarial_tables", validate=False
)
if df_rec.is_empty():
    print("File could not be recovered cleanly — check warnings above.")

# Recover original unscaled values (reverses any decrement_scale_factor applied at
# build time — useful when the table was stored with per-mille or per-ten-thousand rates).
original_df = tb2.get_unscaled_data()
```

### Cloning and modifying an existing table

Every `LifeTable`, `DisabilityTable`, or `ExitTable` exposes its loaded data through
its `.table` attribute, which is a `TableSource` instance.
`TableSource.to_payload()` returns a serialisable `dict` with a `"data"` key
containing a `{column: list}` mapping (starting at `start_age`, without internal
zero-padding rows) and all metadata fields at the top level.  Pass this dict directly
to `TableBuilder.from_payload()`.  This is the recommended way to clone a bundled or
custom table and save a modified variant:

```python
from lactuca import LifeTable, TableBuilder

lt = LifeTable("PASEM2010", "m", interest_rate=0.03)
payload = lt.table.to_payload()

# payload['data'] is a dict of column → list, starting at start_age.
# Modify freely:
payload["table_name"]  = "PASEM2010_adjusted"
payload["description"] = "PASEM 2010 with adjusted omega."
payload["omega"]       = 100

# Truncate data vectors to match the new omega
new_length = payload["omega"] - payload["start_age"] + 1
for col in payload["data"]:
    payload["data"][col] = payload["data"][col][:new_length]

tb_clone = TableBuilder.from_payload(payload)
tb_clone.save(path="actuarial_tables", overwrite=True)
```

:::{note}
The `mi_structure` key is **intentionally absent** from `to_payload()` output.
`TableBuilder` infers it automatically from column names; do not add it to the payload
dict manually — it is silently ignored by `from_payload()`.
:::

---

## `TableBuilder` parameter reference

| Parameter | Type | Required | Description |
|-----------|------|----------|---------|
| `data` | `dict`, `pd.DataFrame`, or `pl.DataFrame` | yes | Table data; column names follow the conventions in *Column names and input formats* above |
| `table_name` | `str` | yes | Short identifier; also used as the default filename stem |
| `table_type` | `"life"`, `"disability"`, or `"exit"` | yes | Determines the required column prefix (`qx_`, `ix_`, or `ox_`) |
| `generational` | `bool` | no | `True` if the table includes improvement-factor columns (`mi_*`) |
| `base_year` | `int` | if generational | Reference year $t_0$ for the improvement projection |
| `generational_formula_type` | `str` | if generational | One of `"exponential_improvement"`, `"linear_improvement"`, `"discrete_improvement"`, `"projected_improvement"`; see {doc}`tables_taxonomy` |
| `grid_years` | `list[int]` or `None` | no | Calendar years present in the year-indexed MI grid; inferred automatically from column names when `None` |
| `start_age` | `int` | no | First age represented in the data (default 0); ages below it are padded with zeros internally |
| `omega` | `int` | no | Terminal (maximum) age; inferred from data length when omitted |
| `decrement_scale_factor` | `int` — power of 10 (1, 10, 100, 1000, …) | no | The engine divides decrement columns by this factor after loading; use when values are stored as per-mille, per-ten-thousand, etc. |
| `mi_scale_factor` | `int` — power of 10 (1, 10, 100, 1000, …) | no | Same as `decrement_scale_factor` but applied to `mi_*` improvement-factor columns |
| `sex_independent` | `bool` | no | `True` when the table has a single set of rates valid for both sexes; the engine mirrors the single-sex column to the other sex at load time; does **not** create a `qx_u` column — `valid_sexes` remains `['f', 'm']` and `sex='u'` still requires `unisex_blend` |
| `select` | `bool` | no | `True` for select-ultimate tables |
| `select_period` | `int` | if select | Count of numbered select-duration columns per sex; must be ≥ 1. Columns are named `_s{sd}` … `_s{sd+N−1}` plus `_ult`, where `sd` (start duration) is auto-detected: 0 for CMI/UK-style tables (e.g. AM92/AF92), 1 for most others. See *Select-ultimate tables* above. |
| `description` | `str` | no | Free-text description stored in the `.ltk` metadata |

See {class}`lactuca.TableBuilder` for the full API reference.

## See also

- {doc}`bundled_tables` — list of available bundled tables
- {doc}`using_tables` — loading, vectorial creation, and dynamic cohort assignment
- {doc}`tables_taxonomy` — Table Taxonomy: temporal and structural classification
- {doc}`modifying_decrements` — adjusting $q_x$ rates after loading

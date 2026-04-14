# Bundled Actuarial Tables

Lactuca ships a curated collection of life, disability and exit actuarial tables stored as
`.ltk` binary files.  This page explains how to manage the bundled tables and provides a
complete reference for every table in the package.

- Install tables once on disk with `Tables.install()` (see [Managing bundled tables](#managing-bundled-tables) below).
- Browse the full catalogue in the [Table reference](#table-reference) section.
- Build a custom table with `TableBuilder` (see {doc}`building_tables`).

:::{warning}
**Data accuracy disclaimer.** The actuarial tables bundled with this package are provided
as a convenience and are believed to be faithful transcriptions of their original published
sources.  They are, however, supplied **without any warranty — express or implied — as to
their accuracy, completeness, or fitness for a particular purpose.**

It is the user's sole responsibility to:

- verify each table against the official published source before relying on it;
- assess its suitability for the intended application and jurisdiction;
- comply with any applicable regulatory requirements.

The author accepts no liability for errors or omissions in the table data, nor for any
decision made in reliance on it.
:::

(managing-bundled-tables)=
## Managing bundled tables

Tables are bundled with the package but must be written to disk as `.ltk` files before use.
The `Tables` class in `lactuca.tables.data` handles all install and discovery operations.

### Installing tables

:::{note}
For generational tables, the `cohort` parameter (birth year) must be an integer in the
range **1900 – current year**.  Values outside this range raise a `ValueError` at
construction time.
:::

```python
from lactuca.tables.data import Tables

# Install all bundled tables to the default directory (Config.tables_path)
Tables.install()

# Install a single table by its Python variable name
Tables.install("PER2020_Ind_1o")

# Install multiple tables at once (list or tuple)
Tables.install(["PER2020_Ind_1o", "PASEM2020_Rel_1o", "DAV2004R_Agg_2o"])

# Install to a custom directory for this call only (overrides Config.tables_path)
Tables.install(tables_path="/srv/actuarial/tables")

# Overwrite existing .ltk files (default: overwrite=False, skips existing)
Tables.install("PER2020_Ind_1o", overwrite=True)

# The return value depends on the call form:
# - single string → bool (True = installed, False = skipped)
# - list or None  → dict[str, bool] mapping each filename to its outcome
result = Tables.install(["PER2020_Ind_1o", "PASEM2020_Rel_1o"])
print(result)   # {'PER2020_Ind_1o.ltk': True, 'PASEM2020_Rel_1o.ltk': False}
                # False means the file already existed and was not overwritten
```

### Discovering available and missing tables

```python
from lactuca.tables.data import Tables

# List all installable filenames (does not check disk — returns all known tables)
available = Tables.list_available()
print(available[:3])   # ['PER2020_Col_2o.ltk', 'PER2020_Col_1o.ltk', 'PER2020_Ind_2o.ltk']
                       # order reflects internal registration order; may vary across releases

# List .ltk filenames not yet present in Config.tables_path
missing = Tables.list_missing()
print(missing)         # [] when all tables are installed, or e.g. ['IASS90.ltk']

# Check against a specific directory instead of the configured default
missing_custom = Tables.list_missing(tables_path="/srv/actuarial/tables")

# Install only the missing tables — safe to call repeatedly (idempotent)
result = Tables.install_missing()      # dict[str, bool] — only newly installed tables appear
print(result)          # {'IASS90.ltk': True}  — {} when all tables were already present
```

### Accessing a single table entry

```python
from lactuca.tables.data import Tables

# Retrieve a TableEntry by Python variable name (returns None if not found)
entry = Tables.get("PASEM2020_Rel_1o")
if entry is not None:
    print(entry.varname)               # 'PASEM2020_Rel_1o'  (also the .ltk file stem)
    print(entry.payload["table_name"]) # 'PASEM 2020 Relativo 1er orden'
    entry.install()                    # install this table individually
```

The default install directory is `actuarial_tables/` at the project root, controlled by
`Config.tables_path`.  Each table is saved under its Python variable name; for example,
`PER2020_Ind_1o` is saved as `PER2020_Ind_1o.ltk`.

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `items` | `str`, `list[str]`, or `None` | `None` | Table(s) to install; `None` installs all |
| `filename` | `str` or `None` | `None` | Override the on-disk filename for single-table installs (ignored for batch) |
| `tables_path` | `str` or `None` | `None` | Override install directory for this call |
| `overwrite` | `bool` | `False` | Replace existing `.ltk` files |
| `strict` | `bool` | `True` | Raise on validation errors; `False` emits warnings |

See {doc}`configuration` for how to change the default install path.

## Need a table that is not listed here?

If the table you need is not bundled with the package, you have two options:

**Request it.** Open a feature request on the {doc}`Support page <../support>` describing the
table (name, source, jurisdiction, and intended use), and attach the data in a readable
format — plain text, CSV, Excel, or any other tabular form.  Requests without accompanying
data cannot be prioritised.  If the table is publicly available and relevant to the
actuarial community, it may be added to a future release.

**Build it yourself.** Any table whose decrement structure fits one of the categories
described in {doc}`tables_taxonomy` — aggregate or select-ultimate, static or generational
— can be created with `TableBuilder` and saved as a `.ltk` file.  See
{doc}`building_tables` for a step-by-step guide.

(table-reference)=
## Table reference

Column guide:

- **File** — Python variable name, also the `.ltk` file stem (e.g. `PER2020_Ind_1o.ltk`).
- **Sex** — sex coverage: `m/f` = both sexes, `m` = male only, `f` = female only, `u` = unisex.
- **Country** — origin country or publishing organisation.
- **Type** — taxonomy code (see {doc}`tables_taxonomy`): combines the class
  (`life` / `disability` / `exit`) with the temporal–structure cell
  (`agg–static`, `agg–gen-a`, `agg–gen-b`, `agg–gen-c`, `sel–static`, `sel–gen-a`, `sel–gen-b`).
- **Order** — actuarial loading: `1st` = prudential (pricing/reserves); `2nd` = best-estimate;
  `—` = not applicable.

### Life tables

| File | Sex | Country | Type | Order |
|------|-----|---------|------|-------|
| `PER2020_Ind_2o` | m/f | Spain (DGSFP) | life · agg–gen-a | 2nd |
| `PER2020_Ind_1o` | m/f | Spain (DGSFP) | life · agg–gen-a | 1st |
| `PER2020_Col_2o` | m/f | Spain (DGSFP) | life · agg–gen-a | 2nd |
| `PER2020_Col_1o` | m/f | Spain (DGSFP) | life · agg–gen-a | 1st |
| `PASEM2020_Gen_2o` | m/f | Spain (DGSFP) | life · agg–static | 2nd |
| `PASEM2020_Dec_2o` | m/f | Spain (DGSFP) | life · agg–static | 2nd |
| `PASEM2020_Rel_1o` | m/f | Spain (DGSFP) | life · agg–static | 1st |
| `PASEM2020_NoRel_1o` | m/f | Spain (DGSFP) | life · agg–static | 1st |
| `PASEM2020_Dec_1o` | m/f | Spain (DGSFP) | life · agg–static | 1st |
| `PASEM2010` | m/f | Spain (DGSFP) | life · agg–static | — |
| `DAV2004R_Agg_2o` | m/f | Germany (DAV) | life · agg–gen-b | 2nd |
| `DAV2004R_Agg_1o` | m/f | Germany (DAV) | life · agg–gen-a | 1st |
| `DAV2004R_SelUlt_2o` | m/f | Germany (DAV) | life · sel–gen-b | 2nd |
| `DAV2004R_SelUlt_1o` | m/f | Germany (DAV) | life · sel–gen-a | 1st |
| `GAM71` | m/f | USA (SOA) | life · agg–static | — |
| `GAM83` | m/f | USA (SOA) | life · agg–static | — |
| `GAM94_AA` | m/f | USA (SOA) | life · agg–gen-a † | — |
| `AM92_AF92` | m/f | UK (CMIB) | life · sel–static ‡‡ | — |
| `CB_H_2020` | m | Chile (CMF) | life · agg–gen-c | — |
| `MI_H_2020` | m | Chile (CMF) | life · agg–gen-c | — |
| `RV_M_2020` | f | Chile (CMF) | life · agg–gen-c | — |
| `MI_M_2020` | f | Chile (CMF) | life · agg–gen-c | — |
| `B_M_2020` | f | Chile (CMF) | life · agg–gen-c | — |

† `GAM94_AA` applies the discrete Scale AA improvement formula (flat age-indexed factors), placing it
structurally in the same class as type-a tables.

‡‡ `AM92_AF92` is a static select-ultimate table using the CMI/UK Duration-0 convention
(`start_duration=0`): the first select year is `duration=0` (unlike most tables where it is
`duration=1`).

### Disability tables

| File | Sex | Country | Type | Order |
|------|-----|---------|------|-------|
| `PEAI2007_IAP_Ind` | m/f | Spain (DGSFP) | disability · agg–static | — |
| `PEAI2007_IAP_Col` | m/f | Spain (DGSFP) | disability · agg–static | — |
| `IASS90` | u ‡ | Spain (SS) | disability · agg–static | — |
| `SS90TOT` | u ‡ | Spain (SS) | disability · agg–static | — |
| `SS90ABS` | u ‡ | Spain (SS) | disability · agg–static | — |

‡ `sex_independent = True`: these tables publish a single rate vector shared by both sexes.
Passing `sex='m'` or `sex='f'` both return the same rates.  The `unisex_blend` parameter
has no effect on these tables.

### Development and test tables

The tables below are synthetic and are intended only for unit testing and development.
They are not suitable for production actuarial calculations.

| File | Sex | Type | Notes |
|------|-----|------|-------|
| `Dummy_qx0` | m/f | life · agg–static | All-zero mortality; verifies annuity = financial annuity |
| `DummyLIFE_1Sm` | m | life · agg–static | Static life table, male only |
| `DummyLIFE_1Sf` | f | life · agg–static | Static life table, female only |
| `DummyLIFE_LinearGen` | m/f | life · agg–gen-a | Synthetic generational life table — linear improvement formula; synthetic, not suitable for production |
| `DummyLIFE_DiscreteGen` | m/f | life · agg–gen-a | Synthetic generational life table — discrete improvement formula; synthetic, not suitable for production |
| `DummyLIFE_ProjectedGen` | m/f | life · agg–gen-c | Synthetic generational life table — projected improvement formula; synthetic, not suitable for production |
| `DummyLIFE_Select` | m/f | life · sel–static | Synthetic select-ultimate life table, select_period=2; synthetic, not suitable for production |
| `DummyLIFE_SelectGen_c` | m/f | life · sel–gen-c | Synthetic select-ultimate life table — projected improvement formula, select_period=2, ages 0–100; synthetic, not suitable for production |
| `DummyLIFE_SelectGen_d` | m/f | life · sel–gen-d | Synthetic select-ultimate life table — per-duration exponential MI, select_period=2, ages 0–100; synthetic, not suitable for production |
| `DummyLIFE_Unisex` | u | life · agg–static | Synthetic sex-independent life table; synthetic, not suitable for production |
| `DummySD2015` | m/f | disability · agg–static | Synthetic disability table |
| `DummySD2015Gen` | m/f | disability · agg–gen-a | Synthetic generational disability table |
| `DummySD_LinearGen` | m/f | disability · agg–gen-a | Synthetic generational disability table — linear improvement formula, ages 20–65; synthetic, not suitable for production |
| `DummySD_DiscreteGen` | m/f | disability · agg–gen-a | Synthetic generational disability table — discrete improvement formula, ages 20–65; synthetic, not suitable for production |
| `DummySD_Gen2o` | m/f | disability · agg–gen-b | Synthetic generational disability table — year-indexed exponential improvement, ages 20–65; synthetic, not suitable for production |
| `DummySD_ProjectedGen` | m/f | disability · agg–gen-c | Synthetic generational disability table — projected improvement formula, ages 20–65; synthetic, not suitable for production |
| `DummySD_Select` | m/f | disability · sel–static | Synthetic select-ultimate disability table, ages 20–65, select_period=2; synthetic, not suitable for production |
| `DummySD_SelectGen_a` | m/f | disability · sel–gen-a | Synthetic select-ultimate disability table — constant exponential MI, uniform across select durations, select_period=2, ages 20–65; synthetic, not suitable for production |
| `DummySD_SelectGen_b` | m/f | disability · sel–gen-b | Synthetic select-ultimate disability table — year-indexed MI, uniform across select durations, select_period=2, ages 20–65; synthetic, not suitable for production |
| `DummySD_SelectGen_c` | m/f | disability · sel–gen-c | Synthetic select-ultimate disability table — projected improvement formula, select_period=2, ages 20–65; synthetic, not suitable for production |
| `DummySD_SelectGen_d` | m/f | disability · sel–gen-d | Synthetic select-ultimate disability table — per-duration exponential MI, select_period=2, ages 20–65; synthetic, not suitable for production |
| `DummySD_Unisex` | u | disability · agg–static | Synthetic sex-independent disability table, ages 20–65; synthetic, not suitable for production |
| `DummyEXIT` | m/f | exit · agg–static | Synthetic exit/withdrawal table |
| `DummyEXIT_Gen` | m/f | exit · agg–gen-a | Synthetic generational exit table |
| `DummyEXIT_LinearGen` | m/f | exit · agg–gen-a | Synthetic generational exit table — linear improvement formula; synthetic, not suitable for production |
| `DummyEXIT_DiscreteGen` | m/f | exit · agg–gen-a | Synthetic generational exit table — discrete improvement formula; synthetic, not suitable for production |
| `DummyEXIT_Gen2o` | m/f | exit · agg–gen-b | Synthetic generational exit table — year-indexed exponential improvement, ages 0–100; synthetic, not suitable for production |
| `DummyEXIT_ProjectedGen` | m/f | exit · agg–gen-c | Synthetic generational exit table — projected improvement formula; synthetic, not suitable for production |
| `DummyEXIT_Select` | m/f | exit · sel–static | Synthetic select-ultimate exit table, select_period=2; synthetic, not suitable for production |
| `DummyEXIT_SelectGen_a` | m/f | exit · sel–gen-a | Synthetic select-ultimate exit table — constant exponential MI, uniform across select durations, select_period=2, ages 0–100; synthetic, not suitable for production |
| `DummyEXIT_SelectGen_b` | m/f | exit · sel–gen-b | Synthetic select-ultimate exit table — year-indexed MI, uniform across select durations, select_period=2, ages 0–100; synthetic, not suitable for production |
| `DummyEXIT_SelectGen_c` | m/f | exit · sel–gen-c | Synthetic select-ultimate exit table — projected improvement formula, select_period=2, ages 0–100; synthetic, not suitable for production |
| `DummyEXIT_SelectGen_d` | m/f | exit · sel–gen-d | Synthetic select-ultimate exit table — per-duration exponential MI, select_period=2, ages 0–100; synthetic, not suitable for production |
| `DummyEXIT_Unisex` | u | exit · agg–static | Synthetic sex-independent exit table; synthetic, not suitable for production |

## See also

- {doc}`using_tables` — how to load these tables into actuarial objects
- {doc}`tables_taxonomy` — Table Taxonomy: temporal and structural classification
- {doc}`mortality_improvement` — how generational improvement factors work
- {doc}`building_tables` — create custom `.ltk` files with `TableBuilder`

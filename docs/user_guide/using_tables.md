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

## Vectorial creation

For small families or scenario analysis (up to 25 instances), pass a sequence for `sex`,
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

Parameters are aligned **element-wise across all three** (`sex`, `cohort`, `duration`), not
as a Cartesian product.  Two sexes and two cohorts produce **2** instances
(male→1960, female→1963), not 4.  All three can vary together:

```python
from lactuca import LifeTable

# Length-2 sequences → 2 instances:
# instance 0: (male,   cohort=1960, duration=1)
# instance 1: (female, cohort=1963, duration=2)
lt0, lt1 = LifeTable("DAV2004R_SelUlt_1o", ("m", "f"), cohort=(1960, 1963), duration=(1, 2))
```

To enumerate all combinations (e.g. 2 sexes × 2 cohorts = 4 tables), call `LifeTable`
separately for each pair.

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

| `sex` length | `cohort` length | `duration` length | Instances created |
|---|---|---|---|
| 1 | 1 | 1 | 1 (plain scalar case) |
| N | 1 | 1 | N (`cohort`/`duration` replicated) |
| 1 | N | 1 | N (`sex`/`duration` replicated) |
| 1 | 1 | N | N (`sex`/`cohort` replicated) |
| N | N | N | N (one-to-one alignment) |

Mismatched lengths greater than 1 raise `ValueError`.  Each parameter can be passed as
a `tuple` or a `list` — both are accepted interchangeably.

:::{warning}
Vectorial creation is limited to 25 instances.  For large portfolios see
{ref}`bulk-portfolios` below.
:::

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

---

**Static tables** (no cohort or duration): create one instance and reuse for all ages:

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
- {doc}`building_tables` — create custom `.ltk` files with `TableBuilder`
- {doc}`bundled_tables` — list of available bundled tables
- {doc}`mortality_improvement` — generational improvement factors and cohort projection
- {doc}`modifying_decrements` — how to adjust $q_x$ rates
- {doc}`tables_taxonomy` — Table Taxonomy: temporal and structural classification
- {doc}`decimals_rounding` — full decimal precision reference


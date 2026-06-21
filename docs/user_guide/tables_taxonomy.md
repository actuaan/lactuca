# Table Taxonomy

Tables in Lactuca are classified along three orthogonal dimensions:

1. **Table type** — *Life* ({class}`lactuca.LifeTable`), *Disability*
   ({class}`lactuca.DisabilityTable`), or *Exit* ({class}`lactuca.ExitTable`).
   This determines which Python class is used to instantiate the table.
2. **Insured structure** — *Aggregate* vs. *Select-Ultimate*.
3. **Temporal nature** — *Static* (no projection) vs. *Generational* (with improvement factors).

Dimensions 2 and 3, together with the mathematical formula used to project rates, define the
**nine category types** described below (Gen (d) applies only to Select-Ultimate tables).  Every included table belongs to exactly one category
and one table type.

Each instance is uniquely identified at runtime by a {class}`lactuca.TableKey`
(`table_name`, `sex`, `cohort`, `duration`, `unisex_blend`) — see
{doc}`using_tables` for the vectorial constructor and dict-return patterns.

---

::::{grid} 1 1 1 1
:gutter: 3

:::{grid-item-card} ① Table type

| Type | Python class | Decrement columns |
|---|---|---|
| Life | `LifeTable` | `qx_m`, `qx_f` |
| Disability | `DisabilityTable` | `ix_m`, `ix_f` |
| Exit | `ExitTable` | `ox_m`, `ox_f` |

:::

:::{grid-item-card} ② Insured structure × Temporal nature

| Category | Aggregate | Select-Ultimate |
|---|---|---|
| **Static** | rates loaded as-is; `cohort` not required | rates by select duration; `cohort` not required |
| **Generational (a)** | time-constant MI per age (`mi_m`/`mi_f`) — exponential / linear / discrete | time-constant MI, uniform across all durations — exponential / linear / discrete |
| **Generational (b)** | year-indexed MI grid (`mi_m_YYYY`) — exponential / linear / discrete | year-indexed MI grid, uniform across durations — exponential / linear / discrete |
| **Generational (c)** | year-indexed MI grid — cumulative product projection | year-indexed MI grid — cumulative product projection |
| **Generational (d)** | *(not applicable)* | independent MI vector per duration — exponential / linear / discrete |

:::

::::

Gen (d) is not applicable to aggregate tables because per-duration MI factors presuppose rates
that vary by duration since underwriting — a concept that only exists in select-ultimate tables.

---

## Aggregate – Static

**Columns**: `qx_m`, `qx_f` / `ix_m`, `ix_f` / `ox_m`, `ox_f` (raw annual decrement rates,
where `m` = male and `f` = female).  No improvement factors.

The decrement vector is loaded as-is from the `.ltk` file.  No `cohort` parameter is accepted.

```python
from lactuca import LifeTable, DisabilityTable, ExitTable

# Life table
pasem = LifeTable("PASEM2020_Rel_1o", "m")

# Disability table (individual business)
peai_ind = DisabilityTable("PEAI2007_IAP_Ind", "m")

# Sex-independent disability table (same rates for "m" and "f")
iass90 = DisabilityTable("IASS90", "m")

# Exit table
exit_t = ExitTable("DummyEXIT", "m")
```

---

## Aggregate – Generational

**Columns**: `qx_m`, `qx_f` plus **MI (Mortality Improvement)** factors `mi_*`.

MI factors quantify the annual reduction of mortality rates over time.  Column names starting
with `mi_` carry these factors; `mi_m` / `mi_f` hold a constant vector per age and sex,
while `mi_m_YYYY` / `mi_f_YYYY` hold values indexed by calendar year.  See
{doc}`mortality_improvement` for the full explanation of each formula and how MI is applied.

A `cohort` (birth year) is mandatory.  For each age $x$ the projected rate is evaluated at
the calendar year $t = \text{cohort} + x$.  Sub-types (a)–(c) differ in how the MI factor
is structured and in the projection formula applied.

### a) Constant improvement factors — cohort projection

The improvement factor is a single age-vector per sex (`mi_m`, `mi_f`), constant across
calendar years.  Three projection formulas are available:

| Formula | Projected rate $q_{x,t}$ |
|---|---|
| `exponential_improvement` | $q_{x,0} \cdot e^{-\lambda_x(t-t_0)}$ |
| `linear_improvement` | $q_{x,0} - mi_x\,(t-t_0)$ |
| `discrete_improvement` | $q_{x,0} \cdot (1-AA_x)^{t-t_0}$ |

```python
from lactuca import LifeTable, DisabilityTable, ExitTable

# Life — exponential improvement (real table)
per = LifeTable("PER2020_Ind_1o", "m", cohort=1969)
# Life — discrete improvement (real table)
gam = LifeTable("GAM94_AA", "m", cohort=1955)
# Life — linear improvement (no real table available)
lin = LifeTable("DummyLIFE_LinearGen", "m", cohort=1970)

# Disability — no real generational disability table available
sd = DisabilityTable("DummySD2015Gen", "m", cohort=1970)

# Exit — no real generational exit table available
ex = ExitTable("DummyEXIT_Gen", "m", cohort=1970)
```

### b) Year-indexed improvement grid — cohort projection

Improvement factors are provided as an annual grid: columns `mi_m_YYYY`, `mi_f_YYYY`
(`YYYY` ∈ [1900, 2200]).  Three projection formulas are available
(`exponential_improvement`, `linear_improvement`, `discrete_improvement`).  For each age
$x$ the calendar year $t = \text{cohort} + x$ is mapped to the nearest available grid
column (years outside the grid are clamped to its boundary).

```python
from lactuca import LifeTable, DisabilityTable, ExitTable

# Life
dav = LifeTable("DAV2004R_Agg_2o", "m", cohort=1960)

# Disability — year-indexed exponential improvement
sd_g2o = DisabilityTable("DummySD_Gen2o", "m", cohort=1970)
# Exit — year-indexed exponential improvement
ex_g2o = ExitTable("DummyEXIT_Gen2o", "m", cohort=1970)
```

### c) Year-indexed improvement grid — cumulative product cohort projection

The improvement factor $AA_{x,t}$ (`projected_improvement`) is indexed by calendar year
(columns `mi_m_YYYY`, `mi_f_YYYY`).  The projected rate is the base rate multiplied by the
cumulative product of annual reduction factors along the insured's diagonal:

$$
q_{x,\,\text{cohort}+x} = q_{x,\,t_0}
  \cdot \prod_{t=t_0+1}^{\text{cohort}+x} \bigl(1 - AA_{x,t}\bigr)
$$

For ages where $\text{cohort} + x \leq t_0$ the base rate is returned unchanged.  Beyond
the last year in the grid the last available factor is used (flat extrapolation).  The `cohort`
parameter is the only projection input exposed in the public API.

```python
from lactuca import LifeTable, DisabilityTable, ExitTable

# Male-only table
cb = LifeTable("CB_H_2020", "m", cohort=1975)

# Female-only table
rv = LifeTable("RV_M_2020", "f", cohort=1975)

print(rv.metadata['generational_formula_type'])

# Disability with projected improvement
sd_proj = DisabilityTable("DummySD_ProjectedGen",  "m", cohort=1975)
# Exit with projected improvement
ex_proj = ExitTable("DummyEXIT_ProjectedGen", "m", cohort=1975)
```

---

## Select-Ultimate tables: actuarial context

**Why select tables exist.** When a life insurance policy is issued, the insured has
recently passed underwriting — a medical examination or a declaration of good health.
Freshly underwritten lives are, on average, healthier than a random individual of the
same age drawn from the general population: this is the **selection effect**. Over the
following years (the *select period*) the advantage wears off as the initial cohort
ages and any surviving impaired lives re-enter the pool. Once the select period has
elapsed the insured is treated as having reverted to *ultimate* (population-level)
mortality.

**When to use `duration` vs `"ult"` in reserves.** For a contract that has been in
force $d$ full years since inception, pass `duration=d` while $d$ is within the select
period; once the select period is exhausted, pass `duration="ult"`. Statutory reserves
for recently underwritten lives must use select mortality rates if the table provides
them; using ultimate rates *before* the select period ends overstates projected
mortality and understates the reserve, which is non-conservative.

## Select-Ultimate — duration indexing

All select-ultimate tables use an integer `duration` parameter at construction time to select
the appropriate mortality column.  Most tables index select columns starting at **duration 1**
(`qx_m_s1`, `qx_m_s2`, …`qx_m_s{sp}`).  A small number of tables — notably those following the
CMI / UK convention, such as AM92 and AF92 — index from **duration 0**
(`qx_m_s0`, `qx_m_s1`, …`qx_m_s{sp}`).  The `start_duration` attribute of `TableSource`
reports the minimum valid duration for a given table (`1` for the standard convention, `0`
for CMI/UK-style tables).  Pass the corresponding minimum duration (or any integer up to
`select_period − 1 + start_duration`) when constructing the table:

```python
from lactuca import LifeTable

# Standard convention (duration-1): select columns named qx_m_s1, qx_m_s2, …
# Example: DAV 2004 R Select-Ultimate (generational — cohort required)
dav_su = LifeTable("DAV2004R_SelUlt_1o", "m", cohort=1960, duration=1)

# CMI/UK convention (duration-0): first select column is qx_m_s0
am92_d0  = LifeTable("AM92_AF92", "m", duration=0)     # freshly underwritten
am92_d1  = LifeTable("AM92_AF92", "m", duration=1)     # 1 year since underwriting
am92_ult = LifeTable("AM92_AF92", "m", duration="ult") # ultimate column

print(am92_d0.table.start_duration)   # 0
```

---

## Select-Ultimate – Static

**Columns**: `qx_m_s1` … `qx_m_s{k}`, `qx_m_ult` (and female equivalents).  No improvement
factors.  No `cohort` parameter is accepted.

The select period (duration since underwriting) determines which column is used.  Once the
select period is exhausted ($d \ge \text{select_period} + \text{start_duration}$) the
ultimate column applies.  The `duration` parameter selects the column at construction time
(see duration indexing conventions above).

```python
from lactuca import LifeTable, DisabilityTable, ExitTable

am92_d0  = LifeTable("AM92_AF92", "m", duration=0)
am92_ult = LifeTable("AM92_AF92", "m", duration="ult")

# Disability select-ultimate (select_period=2, standard convention: duration-1 to "ult")
sd_sel1 = DisabilityTable("DummySD_Select", "m", duration=1)
sd_ult  = DisabilityTable("DummySD_Select", "m", duration="ult")

# Exit select-ultimate (select_period=2, standard convention: duration-1 to "ult")
ex_sel1 = ExitTable("DummyEXIT_Select", "m", duration=1)
ex_ult  = ExitTable("DummyEXIT_Select", "m", duration="ult")
```

---

## Select-Ultimate – Generational

Select-ultimate tables with improvement factors behave like their aggregate counterparts.
The key distinction is whether MI factors are applied **uniformly across all select
durations** (sub-types a–c) or **independently per duration** (sub-type d).

### a) Constant improvement factors — cohort projection

Flat improvement columns (`mi_m`, `mi_f`) applied uniformly to all durations.  Three
projection formulas are available (`exponential_improvement`, `linear_improvement`,
`discrete_improvement`); each duration uses the same MI vector.

```python
from lactuca import LifeTable, DisabilityTable, ExitTable

# Life
dav_su = LifeTable("DAV2004R_SelUlt_1o", "m", cohort=1960, duration=2)

print(dav_su.summary())

# Disability — constant exponential MI, uniform across select durations
sd_sa = DisabilityTable("DummySD_SelectGen_a", "m", cohort=1970, duration=1)
# Exit — constant exponential MI, uniform across select durations
ex_sa = ExitTable("DummyEXIT_SelectGen_a", "m", cohort=1970, duration=1)
```

### b) Year-indexed improvement grid — cohort projection

Annual improvement grid (`mi_m_YYYY`, `mi_f_YYYY`) applied uniformly to all durations.
Three projection formulas are available (`exponential_improvement`, `linear_improvement`,
`discrete_improvement`).  Projection diagonal: $t = \text{cohort} + x$.

```python
from lactuca import LifeTable, DisabilityTable, ExitTable

# Life
dav_su2 = LifeTable("DAV2004R_SelUlt_2o", "m", cohort=1960, duration=2)

print(dav_su2.metadata['generational_formula_type']) # 'exponential_improvement'

# Disability — year-indexed MI, uniform across select durations
sd_sb = DisabilityTable("DummySD_SelectGen_b", "m", cohort=1970, duration=1)
# Exit — year-indexed MI, uniform across select durations
ex_sb = ExitTable("DummyEXIT_SelectGen_b", "m", cohort=1970, duration=1)
```

### c) Year-indexed improvement grid — cumulative product cohort projection

Select-ultimate rates combined with year-indexed improvement $AA_{x,t}$
(`projected_improvement`), projected along each insured's select diagonal
$t = \text{cohort} + x + d$ (where $d$ is the duration since underwriting):

$$
q_{[x]+d,\,\text{cohort}+x+d}
  = q_{[x]+d,\,t_0} \cdot \prod_{t=t_0+1}^{\text{cohort}+x+d} \bigl(1 - AA_{x+d,\,t}\bigr)
$$

Demo tables `DummyLIFE_SelectGen_c`, `DummySD_SelectGen_c`, and `DummyEXIT_SelectGen_c`
are bundled for testing and illustration.

```python
from lactuca import LifeTable, DisabilityTable, ExitTable

# Life — projected improvement, select-ultimate
lt_c = LifeTable("DummyLIFE_SelectGen_c", "m", cohort=1975, duration=1)
# Disability
sd_c = DisabilityTable("DummySD_SelectGen_c", "m", cohort=1975, duration=1)
# Exit
ex_c = ExitTable("DummyEXIT_SelectGen_c", "m", cohort=1975, duration=1)

print(lt_c.metadata['generational_formula_type'])  # 'projected_improvement'
```

### d) Per-duration improvement vectors — cohort projection

Each select duration carries its own improvement vector: columns `mi_m_s1` … `mi_m_s{k}`,
`mi_m_ult` (and female equivalents).  Three projection formulas are available, applied
independently per duration $[d]$, with $t = \text{cohort}+x$:

| Formula | Projected rate $q_{x,t}^{[d]}$ |
|---|---|
| `exponential_improvement` | $q_{x,t_0}^{[d]} \cdot e^{-mi_x^{[d]}(t-t_0)}$ |
| `linear_improvement` | $q_{x,t_0}^{[d]} - mi_x^{[d]}(t-t_0)$ |
| `discrete_improvement` | $q_{x,t_0}^{[d]} \cdot (1-AA_x^{[d]})^{t-t_0}$ |

Demo tables `DummyLIFE_SelectGen_d`, `DummySD_SelectGen_d`, and `DummyEXIT_SelectGen_d`
are bundled for testing and illustration.

```python
from lactuca import LifeTable, DisabilityTable, ExitTable

# Life — per-duration independent exponential MI
lt_d = LifeTable("DummyLIFE_SelectGen_d", "m", cohort=1975, duration=1)
# Disability
sd_d = DisabilityTable("DummySD_SelectGen_d", "m", cohort=1975, duration=1)
# Exit
ex_d = ExitTable("DummyEXIT_SelectGen_d", "m", cohort=1975, duration=1)

print(lt_d.metadata['generational_formula_type'])  # 'exponential_improvement'
```

---

## How each table maps to this taxonomy

For full details — sex coverage, country, and actuarial order — see {doc}`bundled_tables`.

<div class="small">

| Category | Life | Disability | Exit |
|---|---|---|---|
| **Aggregate – Static** | `PASEM2020_Rel_1o`, `PASEM2020_NoRel_1o`, `PASEM2020_Dec_1o`, `PASEM2020_Dec_2o`, `PASEM2020_Gen_2o`, `PASEM2010`, `GAM71`, `GAM83`, `Dummy_qx0`†, `DummyLIFE_1Sm`†, `DummyLIFE_1Sf`†, `DummyLIFE_Unisex`†‡ | `PEAI2007_IAP_Ind`, `PEAI2007_IAP_Col`, `IASS90`‡, `SS90TOT`‡, `SS90ABS`‡, `DummySD2015`†, `DummySD_Unisex`†‡ | `DummyEXIT`†, `DummyEXIT_Unisex`†‡ |
| **Aggregate – Gen (a), exponential** | `PER2020_Ind_1o`, `PER2020_Ind_2o`, `PER2020_Col_1o`, `PER2020_Col_2o`, `DAV2004R_Agg_1o` | `DummySD2015Gen`† | `DummyEXIT_Gen`† |
| **Aggregate – Gen (a), linear** | `DummyLIFE_LinearGen`† | `DummySD_LinearGen`† | `DummyEXIT_LinearGen`† |
| **Aggregate – Gen (a), discrete** | `GAM94_AA`, `DummyLIFE_DiscreteGen`† | `DummySD_DiscreteGen`† | `DummyEXIT_DiscreteGen`† |
| **Aggregate – Gen (b)** | `DAV2004R_Agg_2o` | `DummySD_Gen2o`† | `DummyEXIT_Gen2o`† |
| **Aggregate – Gen (c)** | `CB_H_2020`♂, `MI_H_2020`♂, `RV_M_2020`♀, `B_M_2020`♀, `MI_M_2020`♀, `DummyLIFE_ProjectedGen`† | `DummySD_ProjectedGen`† | `DummyEXIT_ProjectedGen`† |
| **Select-Ultimate – Static** | `AM92_AF92`, `DummyLIFE_Select`† | `DummySD_Select`† | `DummyEXIT_Select`† |
| **Select-Ultimate – Gen (a)** | `DAV2004R_SelUlt_1o` | `DummySD_SelectGen_a`† | `DummyEXIT_SelectGen_a`† |
| **Select-Ultimate – Gen (b)** | `DAV2004R_SelUlt_2o` | `DummySD_SelectGen_b`† | `DummyEXIT_SelectGen_b`† |
| **Select-Ultimate – Gen (c)** | `DummyLIFE_SelectGen_c`† | `DummySD_SelectGen_c`† | `DummyEXIT_SelectGen_c`† |
| **Select-Ultimate – Gen (d)** | `DummyLIFE_SelectGen_d`† | `DummySD_SelectGen_d`† | `DummyEXIT_SelectGen_d`† |

</div>

† Test/demo table.  ‡ `sex_independent = True` (unisex).  ♂ Male-only (`qx_m`). ♀ Female-only (`qx_f`).

:::{warning}
Tables marked with `†` are **synthetic test/demo tables** included for documentation examples
and unit testing only.  Their data is entirely artificial and must not be used for
production actuarial calculations, valuations, reserving, or any other professional purpose.
See {doc}`bundled_tables` for the complete development-table listing.
:::

---

## See also

- {doc}`mortality_improvement` — MI formulas, year-indexed factors, and generational usage
- {doc}`bundled_tables` — full list of bundled table identifiers with metadata
- {doc}`using_tables` — constructor reference, vectorial creation, and dynamic cohort assignment
- {doc}`modifying_decrements` — `modify_qx`, `reset_modifications`
- {doc}`lx_interpolation` — fractional-age interpolation methods

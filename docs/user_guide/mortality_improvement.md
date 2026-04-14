# Mortality Improvement (MI)

## What is mortality improvement?

**Mortality Improvement (MI)** is the systematic reduction of mortality rates over time
caused by advances in medicine, nutrition, sanitation, and living standards.
For a given age $x$, the mortality rate observed in year $t$ is lower than the rate
observed in a base year $t_0$:

$$
q_{x,t} < q_{x,t_0} \qquad \text{for } t > t_0
$$

This improvement is measured by the **MI factor** $\lambda_x$ (or $AA_x$, depending on
the projection formula), which quantifies the annual rate of reduction at each age.

In Lactuca, MI factors are stored in the `.ltk` file alongside the base mortality rates.
All columns whose names start with `mi_` carry MI data:

| Column name | Content |
|---|---|
| `mi_m`, `mi_f` | Constant MI factor by age and sex (scalar per age) |
| `mi_m_YYYY`, `mi_f_YYYY` | MI factor for calendar year `YYYY` by age and sex |

The `cohort` parameter (birth year) is the only projection input required by the public
API.  A person born in year $c$ reaches age $x$ in calendar year $c + x$, so the
relevant mortality rate is the one that applied in that specific year — not the
base-year rate.  Given a cohort, Lactuca evaluates each diagonal $t = \text{cohort} + x$
automatically.

---

## Projection formulas

Lactuca supports four projection formulas.  The formula used by a table is stored in its
`.ltk` metadata field `generational_formula_type` and is selected automatically at load time.
The symbol used for the MI factor varies by publishing body.  Lactuca stores all MI
factors under `mi_m` / `mi_f` columns regardless of formula, but the **interpretation
differs** depending on the formula type:

| Symbol | Interpretation | Formula |
|--------|---------------|---------|
| $\lambda_x$ | Continuous force of mortality improvement | `exponential_improvement` |
| $AA_x$ | Discrete annual reduction rate (SOA convention, e.g. Scale AA) | `discrete_improvement`, `projected_improvement` |
| generic $mi_x$ | Absolute annual improvement (units: mortality points per year) | `linear_improvement` |

These are **not interchangeable notations** for the same quantity.  A value of `0.02`
in an `exponential_improvement` column means $e^{-0.02(t-t_0)} \approx 0.980$ per year;
the same value in a `discrete_improvement` column means $(1-0.02)^{t-t_0} = 0.98^{t-t_0}$
— numerically close but conceptually distinct parametric families.
Other publishing conventions ($F(x)$ in UK CMI tables; *factores de mejora* in Spanish
and Chilean regulations) map to one of these Lactuca formula types depending on the
underlying mathematics used.
### `exponential_improvement`

The most common formula in European and Latin-American regulation:

$$
q_{x,t} = q_{x,t_0} \cdot e^{-\lambda_x \,(t - t_0)}
$$

where $\lambda_x = \text{MI}_x$ is the annual force of improvement at age $x$.

*Bundled tables — constant MI columns* (`mi_m`, `mi_f`): `PER2020_Ind_*`, `PER2020_Col_*`,
`DAV2004R_Agg_1o`, `DAV2004R_SelUlt_1o`.
*Bundled tables — year-indexed columns* (`mi_m_YYYY`, `mi_f_YYYY`): `DAV2004R_Agg_2o`,
`DAV2004R_SelUlt_2o`.

### `linear_improvement`

A simpler linear reduction (flat extrapolation risk for large $t - t_0$):

$$
q_{x,t} = q_{x,t_0} - mi_x \,(t - t_0)
$$

where $mi_x$ is the absolute annual improvement in mortality points at age $x$.

:::{warning}
This formula can produce negative rates if improvement factors are large or the
projection horizon is long.  No automatic floor is applied — a `ValueError` is raised
if any projected probability falls outside $[0, 1]$.  Verify that $mi_x \cdot (t - t_0) \leq q_{x,t_0}$
for all ages and intended cohorts before using this formula in production.
:::

:::{note}
No bundled tables currently use this formula.  It is available for custom tables built
with {class}`lactuca.tables.builder.TableBuilder`; see {doc}`building_tables`.
:::

### `discrete_improvement`

Each year the rate is multiplied by $(1 - AA_x)$, accumulating via a power:

$$
q_{x,t} = q_{x,t_0} \cdot (1 - AA_x)^{\,t - t_0}
$$

where $AA_x$ is the annual percentage reduction factor at age $x$.

*Bundled tables*: `GAM94_AA` (SOA 1994 Group Annuity Mortality, Scale AA).

### `projected_improvement`

A cumulative discrete product applied year by year along the **cohort diagonal**: the
path through the age × calendar-year improvement matrix traced by a person born in year
$c$, reaching age $x$ in calendar year $c + x$.  The annual factor $AA_{x,t}$ may vary
both by age **and** by calendar year:

$$
q_{x,\,\text{cohort}+x} = q_{x,t_0}
  \cdot \prod_{t=t_0+1}^{\text{cohort}+x} \bigl(1 - AA_{x,t}\bigr)
$$

*Bundled tables* (year-indexed columns, `mi_m_YYYY` / `mi_f_YYYY`): `CB_H_2020`,
`MI_H_2020`, `RV_M_2020`, `B_M_2020`, `MI_M_2020` (Chile CMF NCG 305/2023).

```python
from lactuca import LifeTable

# Chilean CMF NCG 305/2023 male table — projected_improvement with annual grid
cb_1960 = LifeTable("CB_H_2020", "m", cohort=1960)
cb_1985 = LifeTable("CB_H_2020", "m", cohort=1985)

# Cohort 1960 reaches age 65 in 2025; cohort 1985 reaches it in 2050
print(f"q65 (cohort 1960): {cb_1960.qx(65):.6f}")
print(f"q65 (cohort 1985): {cb_1985.qx(65):.6f}")   # lower — 25 more years of improvement
```

---

## Constant vs. year-indexed MI factors

### Constant MI factors

The MI factor is a single value per age and sex, independent of calendar year.
Stored as `mi_m` and `mi_f` columns.  Used when the published table provides one
improvement vector without year breakdowns.

**Formulas**: `exponential_improvement`, `linear_improvement`, `discrete_improvement`.

**Example tables**: `PER2020_Ind_1o`, `GAM94_AA`, `DAV2004R_SelUlt_1o`.

### Year-indexed MI factors

The MI factor varies by calendar year: columns `mi_m_YYYY`, `mi_f_YYYY` form an
annual grid.  For cohort $c$ and age $x$, the relevant year is $t = c + x$.

- **Lookup**: the factor for year $t$ is read from the column whose year is the smallest
  grid year $\geq t$ (ceiling lookup via `np.searchsorted`).
- **Before the first grid year**: when `cohort + x` is below the earliest grid year, the
  **first** available factor is used (clamped to the lower grid boundary).
- **Beyond the last grid year**: the last available factor is used (clamped to the
  upper grid boundary).

**Formulas**: `exponential_improvement` (taxonomy sub-type b), `projected_improvement`
(taxonomy sub-type c).  See {doc}`tables_taxonomy` for the full classification.

**Example tables**: `DAV2004R_Agg_2o`, `DAV2004R_SelUlt_2o`, `CB_H_2020`.

### Boundary behavior when `cohort + x < base_year`

The **base year** $t_0$ is the calibration year of the table's published mortality rates,
stored in the `.ltk` metadata.  It is accessible via `lt.table.base_year`.  When the
cohort diagonal falls before $t_0$, behavior depends on the formula:

| Formula | Behavior when $\text{cohort}+x < t_0$ |
|---------|---------------------------------------|
| `exponential_improvement`, `linear_improvement`, `discrete_improvement` | **Base rate returned unchanged** — the exponent or difference is clamped to 0; no back-projection. |
| `projected_improvement` | **Base rate returned unchanged** — the product range is empty (`cum_rf = 1`). |
| Year-indexed (gen-c path) | **First grid year's factor used** — `cal_year` is clamped to `first_grid` before the lookup. |

:::{note}
All formulas return the base-year rate unchanged when `cohort + x < base_year`.
Projecting mortality rates *backward* before $t_0$ — i.e. inflating rather than reducing
$q_x$ — has no actuarial justification: the base table already represents the best
published mortality estimate at $t_0$.

The year-indexed row behaves differently from the scalar-formula rows because there is no
single exponent or difference to clamp to zero: the lookup operates on a column grid, so
the natural boundary is the first available grid year rather than a numeric zero.
:::

---

## Improvement in practice

The examples below show what is specific to MI: how the projected rate varies by cohort
and how to inspect MI metadata on a table instance.  For constructor syntax, vectorial
creation, cohort-guard patterns, and bulk portfolio iteration see {doc}`using_tables`.

:::{note}
Generational tables require `cohort` at construction time.  Omitting it raises a
`ValueError` immediately — the projection year $t = \text{cohort} + x$ cannot be
computed without knowing the year of birth.  Conversely, passing `cohort` to a static
(period) table also raises a `ValueError`.
:::

### Observing the improvement effect

A younger cohort reaches the same age in a later calendar year and therefore benefits
from more accumulated improvement:

```python
from lactuca import LifeTable

lt_1960 = LifeTable("PER2020_Ind_1o", "m", cohort=1960)
lt_1985 = LifeTable("PER2020_Ind_1o", "m", cohort=1985)

# Cohort 1960 reaches age 65 in 2025; cohort 1985 reaches it in 2050
print(f"q65 (cohort 1960): {lt_1960.qx(65):.6f}")
print(f"q65 (cohort 1985): {lt_1985.qx(65):.6f}")   # lower — 25 more years of improvement
```

### Checking MI metadata

```python
from lactuca import LifeTable

lt = LifeTable("PASEM2020_Rel_1o", "m")
print(lt.generational)                    # False — no MI factors

lt_gen = LifeTable("PER2020_Ind_1o", "m", cohort=1969)
print(lt_gen.generational)                # True
print(lt_gen.generational_formula_type)   # 'exponential_improvement'
print(lt_gen.table.base_year)             # 2012 — calibration year of PER2020 base rates
# Equivalent via the underlying TableSource:
print(lt_gen.table.generational_formula_type)
```

---

## MI and the select-ultimate structure

For **select-ultimate** generational tables, the improvement diagonal shifts to
$t = \text{cohort} + x + d$, where $d$ is the select duration since underwriting:

```python
# DAV 2004 R select-ultimate, 1st order — duration 2 (2 years since contract inception)
dav_su = LifeTable("DAV2004R_SelUlt_1o", "m", cohort=1960, duration=2)
```

For `projected_improvement` tables (e.g. Chilean CMF), Lactuca shifts the projection
diagonal to `effective_cohort = cohort + d` so the accumulated product correctly traces
$t = \text{cohort} + x + d$.  For other formulas (e.g. `exponential_improvement`, used
by `DAV2004R_SelUlt_2o`), improvement is applied uniformly across all durations using
$t = \text{cohort} + x$.

:::{note}
No bundled tables combine the select-ultimate structure with `projected_improvement`
(taxonomy sub-type Gen (c) select-ultimate).  The engine supports this combination;
see {doc}`tables_taxonomy` for the full classification.
:::

---

## See also

- {doc}`tables_taxonomy` — full classification of all table types by structure and temporal nature
- {doc}`bundled_tables` — complete catalogue of bundled tables with metadata
- {doc}`using_tables` — constructor reference, vectorial creation, and dynamic cohort assignment
- {doc}`qx_derivation_flow` — how projected $q_x$ values are derived from base rates and MI factors

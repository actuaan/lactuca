# Cookbook

Practical, self-contained recipes for common actuarial tasks using Lactuca.

Each snippet below is copy-paste ready: imports, table identifiers, and parameters match
the bundled catalogue. Run them in an activated Python session (see {doc}`activation`).

Tables are loaded by string identifier and sex code (`'m'`, `'f'`, or `'u'`).
See {doc}`user_guide/bundled_tables` for the full catalogue of available tables.
For bulk portfolio work, see {ref}`bulk-portfolios` in {doc}`user_guide/using_tables`.

## Recipes

| # | Topic | Pattern |
|---|-------|---------|
| 1 | Term life insurance | `Ax(x, n=…)` |
| 2 | Whole-life annuity-due | `äx(x)` |
| 3 | Monthly pension liability | `äx(x, m=12)` + benefit scaling |
| 4 | Joint-life / last-survivor pension | vectorized `LifeTable` + inclusion–exclusion |
| 5 | Interest-rate sensitivity | multi-scenario `InterestRate` |
| 6 | Period vs generational mortality | `cohort=` setter |
| 7 | Tiered benefit schedule | `payment_times` + `tiered_amounts` |
| 8 | Complete expectation of life | `ex` / `ex_continuous` |
| 9 | Portfolio liability (plain Python) | loop + cohort setter |
| 10 | Portfolio liability (Polars) | `iter_rows` + cohort setter |
| 11 | Polars `map_elements` PV column | fixed table, expression pipeline |
| 12 | Portfolio liability (Pandas) | `itertuples` + cohort setter |
| 13 | Portfolio liability (batch API) | functional `äx` + `benefits=` |
| 14 | Net annual premium | `Ax / äx` (equivalence principle) |
| 15 | Deferred pension | `äx(x, d=…)` |
| 16 | Prospective reserve | `ts=` + `Ax - P * äx` |

---

## 1. Price a term life insurance

Net single premium for a 20-year term life insurance on a male aged 45,
using the PASEM2020 individual non-reinsurance table (first order) and a 3 % flat rate.

$$A^1_{x:\overline{n}|} = \sum_{k=0}^{n-1} v^{k+1} \cdot {}_k p_x \cdot q_{x+k}$$

Pass `n=20` to {meth}`~lactuca.tables.LifeTable.Ax` for a temporary insurance; omit `n`
for whole-life.

```python
from lactuca import LifeTable

lt = LifeTable('PASEM2020_NoRel_1o', 'm', interest_rate=0.03)

term_pv = lt.Ax(x=45, n=20)
print(f"Term insurance APV (A^1_45:20) = {term_pv:.6f}")
```

## 2. Whole-life annuity-due

Present value of a unit whole-life annuity-due for a female aged 60,
using the PASEM2020 individual table with reinsurance (first order):

$$\ddot{a}_x^{(m)} = \frac{1}{m} \sum_{k=0}^{\infty} v^{k/m} \cdot {}_{k/m}p_x$$

```python
from lactuca import LifeTable

lt = LifeTable('PASEM2020_Rel_1o', 'f', interest_rate=0.03)

a_due = lt.äx(x=60)
print(f"Whole-life annuity-due (a_dd_60) = {a_due:.6f}")
```

## 3. Pension liability (monthly payments, generational mortality)

Value a pension of €1 000 per month for a male aged 65 born in 1961,
using the generational individual table PER2020 (first order):

```python
from lactuca import LifeTable

lt = LifeTable('PER2020_Ind_1o', 'm', cohort=1961, interest_rate=0.02)

monthly_annuity = lt.äx(x=65, m=12)   # unit annual benefit, m-thly (IAA ä_x^{(12)})
liability = 1_000 * 12 * monthly_annuity   # EUR 1 000/month = EUR 12 000/year
print(f"Pension PV = EUR {liability:,.2f}")
```

:::{note}
`äx(..., m=12)` follows standard actuarial notation: it values **one currency unit
per year** payable in 12 instalments (not one unit per monthly payment).
Scale by the **annual** pension — here `1_000 * 12` — or equivalently
`12_000 * monthly_annuity`.
:::

## 4. Joint-life pension (couple, vectorized instantiation)

Create male and female tables in a single call.
The last-survivor annuity follows from the standard identity:

$$\ddot{a}_{\overline{xy}} = \ddot{a}_x + \ddot{a}_y - \ddot{a}_{xy}$$

```python
from lactuca import LifeTable

# Vectorized instantiation: two LifeTable objects in one call
lt_m, lt_f = LifeTable('PASEM2020_Rel_1o', ('m', 'f'), interest_rate=0.02)

a_x     = lt_m.äx(x=65, m=12)
a_y     = lt_f.äx(x=62, m=12)
a_joint = lt_m.äxy(ages=[65, 62], table_y=lt_f, m=12)   # both survive
a_ls    = a_x + a_y - a_joint                            # last survivor

print(f"Individual (male  65): {a_x:.4f}")
print(f"Individual (female 62): {a_y:.4f}")
print(f"Joint-life (both alive): {a_joint:.4f}")
print(f"Last-survivor:           {a_ls:.4f}")
```

See {doc}`user_guide/joint_life_calculations` for first-death insurances (`Axy`, `Afirst`)
and other derivable joint-life formulas.

## 5. Interest rate sensitivity

Compare annuity values across named interest rate scenarios using a multiscenario
`InterestRate` — the natural pattern for sensitivity analyses and Solvency II stress tests.

**Pattern A — iterate sub-curves.**  Extract each simple scenario and pass it as `ir=`:

```python
from lactuca import LifeTable, InterestRate

ir = InterestRate({
    'base':      0.03,
    'adverse':   0.01,
    'stressed':  0.00,
})

lt = LifeTable('PASEM2020_Gen_2o', 'm')

for name, scenario in ir.scenarios.items():
    value = lt.äx(x=65, ir=scenario)
    print(f"{name:<10}  i = {scenario.rate:.2%}  ->  a_dd_65 = {value:.4f}")
```

**Pattern B — switch `active_scenario` on a shared container.**  Attach the
multi-scenario object once (to `LifeTable` or via `ir=`) and switch scenarios in place:

```python
from lactuca import LifeTable, InterestRate

ir = InterestRate({
    'base':      0.03,
    'adverse':   0.01,
    'stressed':  0.00,
})

lt = LifeTable('PASEM2020_Gen_2o', 'm', interest_rate=ir)

ir.active_scenario = 'base'
bel_base = lt.äx(65)

ir.active_scenario = 'stressed'
bel_stressed = lt.äx(65)

print(f"base: {bel_base:.4f}  stressed: {bel_stressed:.4f}")
```

Both patterns yield the same PV per scenario.  Pattern B avoids allocating separate
`InterestRate` wrappers in a loop; Pattern A makes each scenario explicit at the call site.
See {ref}`interest-rate-scenarios-lifetable` for `copy()` snapshotting and batch semantics.

:::{note}
`scenario.rate` is defined for **constant** sub-curves (as in this example).
For piecewise scenarios, use `scenario.get_rate(t)` or inspect `scenario.rates`.
:::

## 6. Cohort vs. period projection

Compare period and generational annuity values. A 65-year-old in 2026 was born
in 1961. The generational table `PER2020_Ind_2o` embeds projection improvements:

```python
from lactuca import LifeTable

lt_period = LifeTable('PASEM2020_Rel_1o', 'm', interest_rate=0.03)
lt_cohort = LifeTable('PER2020_Ind_2o',   'm', cohort=1961, interest_rate=0.03)

a_period = lt_period.äx(x=65)
a_cohort = lt_cohort.äx(x=65)

print(f"Period annuity:  {a_period:.4f}")
print(f"Cohort annuity:  {a_cohort:.4f}")
print(f"Difference:      {a_cohort - a_period:.4f}")
```

## 7. Custom benefit schedule

Value a pension that pays €12 000/year for 10 years, then €8 000/year thereafter,
using `ax` (annuity-immediate), which supports custom cashflow schedules:

```python
from lactuca import LifeTable, payment_times, tiered_amounts

lt = LifeTable('PASEM2020_Rel_1o', 'm', interest_rate=0.03)

times   = payment_times(n=40, m=1)
amounts = tiered_amounts(times, breakpoints=[10], values=[12_000.0, 8_000.0])

pv = lt.ax(x=65, cashflow_times=times, cashflow_amounts=amounts)
print(f"Custom benefit PV = EUR {pv:,.2f}")
```

## 8. Complete expectation of life

Compute $\mathring{e}_x$ at an integer age and at a fractional age.
`ex_continuous` only accepts fractional ages; pass `65.5`, not `65`:

```python
from lactuca import LifeTable

lt = LifeTable('PASEM2020_Rel_1o', 'm')

ex_int  = lt.ex(65)
ex_frac = lt.ex_continuous(65.5)

print(f"e_65   = {ex_int:.2f} years  (integer age, discrete)")
print(f"e_65.5 = {ex_frac:.2f} years  (fractional age, continuous)")
```

(portfolio-liability)=
## 9. Portfolio liability valuation

Compute the present value of a pension portfolio from a plain Python list of
records — no external dependencies beyond Lactuca.
Ages are derived with `alb`, the `cohort` setter is updated inside the loop
only when it changes, following the bulk-portfolio approach from
{doc}`user_guide/using_tables`.

```python
from lactuca import LifeTable, GrowthRate, alb

VALUATION_DATE = '2026-04-09'

# Pension portfolio: birth date, sex, annual pension,
# term in years (n=None -> whole-life), escalation rate.
portfolio = [
    {'id': 'P001', 'birth': '1955-03-15', 'sex': 'm', 'pension': 18_000, 'n': 25,   'g': 0.020},
    {'id': 'P002', 'birth': '1958-11-22', 'sex': 'f', 'pension': 12_000, 'n': None, 'g': 0.010},
    {'id': 'P003', 'birth': '1950-07-04', 'sex': 'm', 'pension': 24_000, 'n': 20,   'g': 0.025},
    {'id': 'P004', 'birth': '1962-01-30', 'sex': 'f', 'pension':  9_600, 'n': None, 'g': 0.000},
    {'id': 'P005', 'birth': '1957-09-10', 'sex': 'm', 'pension': 15_000, 'n': 28,   'g': 0.015},
]

# Vectorized age and cohort (birth year) derivation
births   = [p['birth'] for p in portfolio]
ages_alb = alb(births, VALUATION_DATE)            # NDArray[float64]
for p, age in zip(portfolio, ages_alb):
    p['age']    = int(age)
    p['cohort'] = int(p['birth'][:4])             # birth year from ISO date string

# One LifeTable per sex; cohort updated per policy inside the loop
tables = {
    'm': LifeTable('PER2020_Ind_1o', 'm', cohort=1950, interest_rate=0.03),
    'f': LifeTable('PER2020_Ind_1o', 'f', cohort=1950, interest_rate=0.03),
}

# Sort by (sex, cohort) to minimise cohort-setter rebuilds
for p in sorted(portfolio, key=lambda r: (r['sex'], r['cohort'])):
    lt = tables[p['sex']]
    if lt.cohort != p['cohort']:
        lt.cohort = p['cohort']
    gr  = GrowthRate(p['g']) if p['g'] else None
    pv  = p['pension'] * lt.äx(x=p['age'], n=p['n'], m=12, gr=gr)
    p['pv'] = round(pv, 2)

total_liability = sum(p['pv'] for p in portfolio)
print(f"{'ID':<6} {'Sex':>3} {'Age':>4} {'Cohort':>6} {'Pension':>10} {'g':>5} {'PV':>14}")
print('-' * 52)
for p in portfolio:
    print(f"{p['id']:<6} {p['sex']:>3} {p['age']:>4} {p['cohort']:>6} "
          f"{p['pension']:>10,.0f} {p['g']:>5.1%} {p['pv']:>14,.2f}")
print('-' * 52)
print(f"{'Total liability':>42}  {total_liability:>14,.2f}")
```

(portfolio-liability-polars)=
## 10. Portfolio liability valuation with Polars

Compute the present value of a pension portfolio loaded as a Polars DataFrame
(simulating a read from Excel or any other tabular source).
Ages are derived with `alb` from the `birth` date column; the `cohort` setter
is updated inside the loop only when it changes, following the bulk-portfolio
approach from {doc}`user_guide/using_tables`.

```python
import polars as pl
from lactuca import LifeTable, GrowthRate, alb

VALUATION_DATE = '2026-04-09'

# In production: df = pl.read_excel("portfolio.xlsx", sheet_name="Policyholders")
df = pl.DataFrame({
    'id':      ['P001', 'P002', 'P003', 'P004', 'P005'],
    'birth':   ['1955-03-15', '1958-11-22', '1950-07-04', '1962-01-30', '1957-09-10'],
    'sex':     ['m', 'f', 'm', 'f', 'm'],
    'pension': [18_000.0, 12_000.0, 24_000.0, 9_600.0, 15_000.0],
    'n':       [25, None, 20, None, 28],         # None -> whole-life annuity
    'g':       [0.020, 0.010, 0.025, 0.000, 0.015],
}).with_columns(
    pl.col('birth').str.to_date()               # parse ISO 8601 strings -> pl.Date
)

# Add age-at-last-birthday (vectorized) and cohort (birth year) columns
ages_alb = alb(df['birth'].to_list(), VALUATION_DATE)   # NDArray[float64]
df = df.with_columns(
    pl.Series('age', ages_alb).cast(pl.Int32),
    pl.col('birth').dt.year().alias('cohort'),
)

# One LifeTable per sex; cohort will be updated inside the loop only when it changes
tables = {
    'm': LifeTable('PER2020_Ind_1o', 'm', cohort=1950, interest_rate=0.03),
    'f': LifeTable('PER2020_Ind_1o', 'f', cohort=1950, interest_rate=0.03),
}

# Sort by (sex, cohort) to minimise cohort-setter rebuilds
df_sorted = df.sort(['sex', 'cohort'])

pv_list = []
for row in df_sorted.iter_rows(named=True):
    lt = tables[row['sex']]
    if lt.cohort != row['cohort']:
        lt.cohort = row['cohort']
    gr  = GrowthRate(row['g']) if row['g'] else None
    pv  = row['pension'] * lt.äx(x=row['age'], n=row['n'], m=12, gr=gr)
    pv_list.append(round(pv, 2))

result = df_sorted.with_columns(pl.Series('pv', pv_list))

print(result.select(['id', 'sex', 'age', 'cohort', 'pension', 'g', 'pv']))
print(f"\nTotal liability:  {result['pv'].sum():>14,.2f}")
```

## 11. Adding a present-value column with Polars `map_elements`

Use Polars' `map_elements` on a struct column to add a `pv` column inline within
an expression pipeline. This pattern is ideal when the table is fixed (single sex,
no cohort updates) and you want to keep the transformation inside a Polars chain.

```python
import polars as pl
from lactuca import LifeTable, GrowthRate

lt = LifeTable('PASEM2020_Rel_1o', 'f', interest_rate=0.03)

df = pl.DataFrame({
    'id':      ['A001', 'A002', 'A003'],
    'age':     [60, 62, 65],
    'pension': [15_000.0, 12_000.0, 20_000.0],
    'n':       [30, 28, 25],
    'g':       [0.02, 0.01, 0.00],
})

result = df.with_columns(
    pl.struct(['age', 'pension', 'n', 'g']).map_elements(
        lambda row: row['pension'] * lt.äx(
            x=row['age'],
            n=row['n'],
            m=12,
            gr=GrowthRate(row['g']) if row['g'] else None,
        ),
        return_dtype=pl.Float64,
    ).alias('pv')
)

print(result)
```

:::{note}
`map_elements` applies a Python function to each element of the struct Series —
it is equivalent to a row-wise loop and does not unlock Polars parallelism.
Use it when you need the result as a Polars expression inside a pipeline
(`with_columns`, `select`, lazy frames, etc.).
For portfolios with cohort updates or multiple tables, prefer `iter_rows(named=True)`
as in {ref}`recipe 10 <portfolio-liability-polars>` (or the pure-Python variant in
{ref}`recipe 9 <portfolio-liability>`).
:::

## 12. Portfolio valuation with Pandas

The Pandas equivalent of recipe 9. Use `itertuples` (faster than `iterrows`) after
sorting by `(sex, cohort)` to minimise cohort-setter rebuilds.

```python
import pandas as pd
from lactuca import LifeTable, GrowthRate, alb

VALUATION_DATE = '2026-04-09'

# In production: df = pd.read_excel("portfolio.xlsx", sheet_name="Policyholders")
df = pd.DataFrame({
    'id':      ['P001', 'P002', 'P003', 'P004', 'P005'],
    'birth':   ['1955-03-15', '1958-11-22', '1950-07-04', '1962-01-30', '1957-09-10'],
    'sex':     ['m', 'f', 'm', 'f', 'm'],
    'pension': [18_000.0, 12_000.0, 24_000.0, 9_600.0, 15_000.0],
    'n':       [25, None, 20, None, 28],         # None -> whole-life annuity
    'g':       [0.020, 0.010, 0.025, 0.000, 0.015],
})
df['birth'] = pd.to_datetime(df['birth'])

# Vectorized age and cohort columns
ages_alb     = alb(df['birth'].tolist(), VALUATION_DATE)   # NDArray[float64]
df['age']    = ages_alb.astype(int)
df['cohort'] = df['birth'].dt.year

# One LifeTable per sex
tables = {
    'm': LifeTable('PER2020_Ind_1o', 'm', cohort=1950, interest_rate=0.03),
    'f': LifeTable('PER2020_Ind_1o', 'f', cohort=1950, interest_rate=0.03),
}

# Sort to minimise cohort-setter rebuilds, then iterate with itertuples
df_sorted = df.sort_values(['sex', 'cohort']).reset_index(drop=True)
pv_list = []
for row in df_sorted.itertuples():
    lt = tables[row.sex]
    if lt.cohort != row.cohort:
        lt.cohort = row.cohort
    gr    = GrowthRate(row.g) if row.g else None
    n_val = None if pd.isna(row.n) else row.n
    pv    = row.pension * lt.äx(x=row.age, n=n_val, m=12, gr=gr)
    pv_list.append(round(pv, 2))

df_sorted['pv'] = pv_list
print(df_sorted[['id', 'sex', 'age', 'cohort', 'pension', 'g', 'pv']].to_string(index=False))
print(f"\nTotal liability:  {df_sorted['pv'].sum():>14,.2f}")
```

:::{note}
`df['birth'].tolist()` on a `datetime64` column yields `pd.Timestamp` objects,
which are a valid date type for `alb`. Passing the list directly avoids any
intermediate conversion.
`itertuples` iterates as named tuples (fields accessed as `row.sex`, `row.age`, etc.)
and is significantly faster than `iterrows` for large DataFrames.
:::

(portfolio-liability-batch)=
## 13. Portfolio liability valuation (batch API)

Vectorized alternative to recipes 9–12: one functional call prices the entire portfolio
without a Python loop over policies. Build one `LifeTable` per unique `(sex, cohort)` pair,
then pass a per-policy table list to {func}`~lactuca.functional.äx` with `benefits=`.

```python
import numpy as np
from lactuca import LifeTable, TableKey, GrowthRate, alb, äx

VALUATION_DATE = '2026-04-09'

portfolio = [
    {'id': 'P001', 'birth': '1955-03-15', 'sex': 'm', 'pension': 18_000, 'n': 25,   'g': 0.020},
    {'id': 'P002', 'birth': '1958-11-22', 'sex': 'f', 'pension': 12_000, 'n': None, 'g': 0.010},
    {'id': 'P003', 'birth': '1950-07-04', 'sex': 'm', 'pension': 24_000, 'n': 20,   'g': 0.025},
    {'id': 'P004', 'birth': '1962-01-30', 'sex': 'f', 'pension':  9_600, 'n': None, 'g': 0.000},
    {'id': 'P005', 'birth': '1957-09-10', 'sex': 'm', 'pension': 15_000, 'n': 28,   'g': 0.015},
]

births   = [p['birth'] for p in portfolio]
ages_alb = alb(births, VALUATION_DATE)
for p, age in zip(portfolio, ages_alb):
    p['age']    = int(age)
    p['cohort'] = int(p['birth'][:4])

# One LifeTable per unique (sex, cohort) — not one per policy row
unique_pairs = sorted({(p['sex'], p['cohort']) for p in portfolio})
tables_by_key = LifeTable(
    'PER2020_Ind_1o',
    [s for s, _ in unique_pairs],
    cohort=[c for _, c in unique_pairs],
    return_dict=True,
    interest_rate=0.03,
)

table_list = [
    tables_by_key[TableKey('PER2020_Ind_1o', p['sex'], p['cohort'])]
    for p in portfolio
]
ages     = [p['age'] for p in portfolio]
n_list   = [p['n'] for p in portfolio]   # None -> whole-life (same as scalar)
gr_list  = [GrowthRate(p['g']) if p['g'] else None for p in portfolio]
benefits = [p['pension'] for p in portfolio]

pv_arr = äx(table_list, ages, n=n_list, m=12, gr=gr_list, benefits=benefits)

for p, pv in zip(portfolio, pv_arr):
    p['pv'] = round(float(pv), 2)

total_liability = float(pv_arr.sum())
print(f"Total liability (batch):  {total_liability:>14,.2f}")
```

:::{important}
In both **scalar** and **batch** mode, `n=None` means whole-life — including
`None` entries in a per-policy `n` list.  You may also write `np.inf` explicitly
in batch vectors; both forms are equivalent.  Pure endowments (`nEx`, …) require
a finite term and reject whole-life sentinels.
See {doc}`user_guide/batch_calculations` for per-policy parameters, `on_error='nan'`,
and aggregate cash-flow patterns.
:::


(net-premium)=
## 14. Net annual premium (equivalence principle)

Under the equivalence principle, the net level annual premium for a unit benefit is the
ratio of the insurance APV to the premium annuity-due APV:

$$P = \frac{A_{x:\overline{n}|}}{\ddot{a}_{x:\overline{n}|}}$$

Complements recipe 1 (term insurance `Ax`); adjust `x`, `n`, and `interest_rate` as needed.

```python
from lactuca import LifeTable

lt = LifeTable('PASEM2020_NoRel_1o', 'm', interest_rate=0.03)

x, n = 45, 20
benefit_pv  = lt.Ax(x=x, n=n)
premium_ann = lt.äx(x=x, n=n)
net_premium = benefit_pv / premium_ann

print(f"Benefit APV  A^1_{x}:{n} = {benefit_pv:.6f}")
print(f"Premium APV  a_dd_{x}:{n} = {premium_ann:.4f}")
print(f"Net annual premium P      = {net_premium:.6f}")
```

(deferred-pension)=
## 15. Deferred pension to retirement age

Value an annual pension of EUR 24 000 starting at age 65 for a member currently aged 52.
Pass the deferment in years as `d=` — payments begin at age `x + d`:

$$\ddot{a}_{x:\overline{n}|}^{(m)} \text{ with deferment } d \quad\Rightarrow\quad {}_d|\ddot{a}_x^{(m)}$$

```python
from lactuca import LifeTable

lt = LifeTable('PASEM2020_Rel_1o', 'm', interest_rate=0.03)

current_age    = 52
retirement_age = 65
defer_years    = retirement_age - current_age

unit_annuity = lt.äx(x=current_age, d=defer_years, m=12)
annual_pension = 24_000
liability = annual_pension * unit_annuity

print(f"Unit deferred annuity (m=12) = {unit_annuity:.4f}")
print(f"Deferred pension PV          = EUR {liability:,.2f}")
```

:::{note}
`d=` is **future** deferment before the first payment. For valuation *after* issue,
use `ts=` instead (see {ref}`recipe 16 <prospective-reserve>` and {doc}`user_guide/deferment`).
Benefit is **annual**; `äx(..., m=12)` uses standard IAA notation (one unit per year
in 12 instalments) — scale by the annual amount directly.
:::

(prospective-reserve)=
## 16. Prospective reserve at policy anniversaries

The prospective reserve at elapsed time $t$ is future benefits minus future net premiums,
both evaluated from attained age $x + t$. In Lactuca, pass the issue age `x`, full term `n`,
and elapsed years `ts=t`:

$${}_t V_x = A_{x+t:\overline{n-t}|} - P \cdot \ddot{a}_{x+t:\overline{n-t}|}$$

```python
from lactuca import LifeTable

lt = LifeTable('PASEM2020_Rel_1o', 'm', interest_rate=0.03)

x, n = 40, 25

# Net level premium at issue (ts = 0)
P = lt.Ax(x, n=n) / lt.äx(x, n=n)
print(f"Net level premium P = {P:.6f}")

# Prospective reserve at selected anniversaries
print(f"\n{'t':>4}  {'A(x+t)':>12}  {'a_dd(x+t)':>12}  {'tV':>12}")
for t in (0, 5, 10, 15, 20, 25):
    At = lt.Ax(x, n=n, ts=t)
    at = lt.äx(x, n=n, ts=t)
    tV = At - P * at
    print(f"{t:>4}  {At:>12.6f}  {at:>12.6f}  {tV:>12.6f}")
```

:::{note}
At `ts=0` the reserve is zero by construction (equivalence principle). At `ts=n` both
benefit and premium APVs are zero, so ${}_n V_x = 0`. See {doc}`user_guide/prospective_reserve`
for fractional `ts`, interaction with `d=`, and joint-life products.
:::


## See also

- {doc}`user_guide/using_tables` — vectorized construction, cohort setter, bulk portfolios
- {doc}`user_guide/batch_calculations` — vectorized alternative to recipes 9–12: pass an age array to a single call for 50–250× speedup
- {doc}`user_guide/building_tables` — create custom `.ltk` files with `TableBuilder`
- {doc}`user_guide/interest_rates_guide` — `InterestRate` construction and scenarios
- {doc}`user_guide/joint_life_calculations` — joint-life annuities, insurances, and derivable formulas
- {doc}`user_guide/deferment` — deferred benefits (`d=`) and distinction from `ts`
- {doc}`user_guide/prospective_reserve` — fractional `ts`, growth with reserves, joint-life
- {doc}`user_guide/irregular_cashflows` — arbitrary cashflow timing

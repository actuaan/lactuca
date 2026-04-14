# Cookbook

Practical, self-contained recipes for common actuarial tasks using Lactuca.

Tables are loaded by string identifier and sex code (`'m'`, `'f'`, or `'u'`).
See {doc}`user_guide/bundled_tables` for the full catalogue of available tables.
For bulk portfolio work, see {ref}`bulk-portfolios` in {doc}`user_guide/using_tables`.

## 1. Price a term life insurance

Net single premium for a 20-year term life insurance on a male aged 45,
using the PASEM2020 individual non-reinsurance table (first order) and a 3 % flat rate.

$$A^1_{x:\overline{n}|} = \sum_{k=0}^{n-1} v^{k+1} \cdot {}_k p_x \cdot q_{x+k}$$

```python
from lactuca import LifeTable

lt = LifeTable('PASEM2020_NoRel_1o', 'm', interest_rate=0.03)

a1 = lt.Ax(x=45, n=20)
print(f"A_45:20 = {a1:.6f}")
```

## 2. Whole-life annuity-due

Present value of a unit whole-life annuity-due for a female aged 60,
using the PASEM2020 individual table with reinsurance (first order):

$$\ddot{a}_x^{(m)} = \frac{1}{m} \sum_{k=0}^{\infty} v^{k/m} \cdot {}_{k/m}p_x$$

```python
from lactuca import LifeTable

lt = LifeTable('PASEM2020_Rel_1o', 'f', interest_rate=0.03)

a = lt.äx(x=60)
print(f"ä_60 = {a:.6f}")
```

## 3. Pension liability (monthly payments, generational mortality)

Value a pension of €1 000 per month for a male aged 65 born in 1961,
using the generational individual table PER2020 (first order):

```python
from lactuca import LifeTable

lt = LifeTable('PER2020_Ind_1o', 'm', cohort=1961, interest_rate=0.02)

monthly_annuity = lt.äx(x=65, m=12)
liability = 1_000 * 12 * monthly_annuity
print(f"Pension PV = €{liability:,.2f}")
```

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
a_joint = lt_m.äxy(ages=[65, 62], table_y=lt_f, m=12)
a_ls    = a_x + a_y - a_joint

print(f"Individual (male  65): {a_x:.4f}")
print(f"Individual (female 62): {a_y:.4f}")
print(f"Joint first-death:      {a_joint:.4f}")
print(f"Last-survivor:          {a_ls:.4f}")
```

## 5. Interest rate sensitivity

Compare annuity values across named interest rate scenarios using a multiscenario
`InterestRate` — the natural pattern for sensitivity analyses and Solvency II stress tests.
Each scenario is activated in turn by setting `active_scenario`:

```python
from lactuca import LifeTable, InterestRate

# Named scenarios: base + two stress levels
ir = InterestRate({
    'base':      0.03,
    'adverse':   0.01,
    'stressed':  0.00,
})

lt = LifeTable('PASEM2020_Gen_2o', 'm')

for name, scenario in ir.scenarios.items():
    value = lt.äx(x=65, ir=scenario)
    print(f"{name:<10}  i = {scenario.rate:.2%}  →  ä_65 = {value:.4f}")
```

## 6. Cohort vs. period projection

Compare period and generational annuity values. A 65-year-old in 2026 was born
in 1961. The generational table `PER2020_Ind_2o` embeds projection improvements:

```python
from lactuca import LifeTable

lt_period = LifeTable('PASEM2020_Rel_1o', 'm', interest_rate=0.03)
lt_cohort  = LifeTable('PER2020_Ind_2o',   'm', cohort=1961, interest_rate=0.03)

a_period = lt_period.äx(x=65)
a_cohort  = lt_cohort.äx(x=65)

print(f"Period annuity:  {a_period:.4f}")
print(f"Cohort annuity:  {a_cohort:.4f}")
print(f"Difference:      {a_cohort - a_period:.4f}")
```

## 7. Custom benefit schedule

Value a pension that pays €12 000/year for 10 years, then €8 000/year thereafter,
using `ax` (annuity-immediate), which supports custom cashflow schedules:

```python
from lactuca import LifeTable, generate_payment_times as gpt, tiered_amounts

lt = LifeTable('PASEM2020_Rel_1o', 'm', interest_rate=0.03)

times   = gpt(n=40, m=1)
amounts = tiered_amounts(times, breakpoints=[10], values=[12_000.0, 8_000.0])

pv = lt.ax(x=65, cashflow_times=times, cashflow_amounts=amounts)
print(f"Custom benefit PV = €{pv:,.2f}")
```

## 8. Complete expectation of life

Compute $\mathring{e}_x$ at an integer age and at a fractional age.
`ex_continuous` only accepts fractional ages; pass `65.5`, not `65`:

```python
from lactuca import LifeTable

lt = LifeTable('PASEM2020_Rel_1o', 'm')

ex_int  = lt.ex(65)
ex_frac = lt.ex_continuous(65.5)

print(f"ė_65   = {ex_int:.2f} years  (integer age)")
print(f"ė_65.5 = {ex_frac:.2f} years  (fractional age)")
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
# term in years (n=None → whole-life), escalation rate.
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
    'n':       [25, None, 20, None, 28],         # None → whole-life annuity
    'g':       [0.020, 0.010, 0.025, 0.000, 0.015],
}).with_columns(
    pl.col('birth').str.to_date()               # parse ISO 8601 strings → pl.Date
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
    'n':       [25, None, 20, None, 28],         # None → whole-life annuity
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

## See also

- {doc}`user_guide/using_tables` — vectorized construction, cohort setter, bulk portfolios
- {doc}`user_guide/building_tables` — create custom `.ltk` files with `TableBuilder`
- {doc}`user_guide/interest_rates_guide` — `InterestRate` construction and scenarios
- {doc}`user_guide/joint_life_calculations` — joint-life annuities, insurances, and derivable formulas
- {doc}`user_guide/irregular_cashflows` — arbitrary cashflow timing

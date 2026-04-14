# Getting Started

This guide walks you through the first steps with Lactuca: installing the library,
loading an actuarial table, and running common probability and valuation calculations.
No prior knowledge of the package is assumed.

## Installation

Lactuca runs on Windows, macOS, and Linux. **Python ≥ 3.10** is required.

```bash
pip install lactuca
```

All other dependencies (NumPy ≥ 2.3, Pandas ≥ 2.3, SciPy ≥ 1.16, Polars ≥ 1.34) are installed automatically.

All actuarial tables included in Lactuca are available immediately after installation —
no separate download is required. See {doc}`bundled_tables` for the complete catalogue.

## Quick example

```python
from lactuca import LifeTable, äx

# First-order individual annuitant table (PER 2020, born 1969)
lt = LifeTable(table_name="PER2020_Ind_1o", sex="m", cohort=1969)

# 15-year temporary life annuity-due (annual payments, 3% interest)
annuity = lt.äx(x=50, n=15, m=1, ir=0.03)
print(f"Life annuity value: {annuity:.4f}")

# The same calculation via the functional API:
annuity = äx(table=lt, x=50, n=15, m=1, ir=0.03)
print(f"Life annuity value: {annuity:.4f}")
```

## How Lactuca is organised

Lactuca is structured as five layers.  As a user you interact exclusively with the
**Public API** layer; the layers below it are handled automatically:

<div style="font-family: monospace; border: 1px solid #ccc; border-radius: 4px; overflow: hidden; margin: 1em 0;">
<table style="border-collapse: collapse; width: 100%; margin: 0;">
<tr style="background: #e3f2fd;">
  <td style="padding: 8px 16px; border-bottom: 1px solid #ccc; font-weight: bold;">Public API</td>
  <td style="padding: 8px 16px; border-bottom: 1px solid #ccc;"><code>LifeTable</code>, <code>DisabilityTable</code>, <code>ExitTable</code>, <code>InterestRate</code>, <code>GrowthRate</code>, <code>Config</code>, functional API (<code>ax</code>, <code>äx</code>, <code>Ax</code>, <code>nEx</code>, …)</td>
</tr>
<tr style="background: #e8f5e9;">
  <td style="padding: 8px 16px; border-bottom: 1px solid #ccc; font-weight: bold;">Calculation layer</td>
  <td style="padding: 8px 16px; border-bottom: 1px solid #ccc;">Annuity, insurance, and endowment dispatchers; interest-rate and growth-rate models; global <code>Config</code> singleton</td>
</tr>
<tr style="background: #f3e5f5;">
  <td style="padding: 8px 16px; border-bottom: 1px solid #ccc; font-weight: bold;">Table layer</td>
  <td style="padding: 8px 16px; border-bottom: 1px solid #ccc;"><code>LifeTable</code> · <code>DisabilityTable</code> · <code>ExitTable</code> — load tables, compute probabilities, apply modifications</td>
</tr>
<tr style="background: #fff3e0;">
  <td style="padding: 8px 16px; border-bottom: 1px solid #ccc; font-weight: bold;">Core layer</td>
  <td style="padding: 8px 16px; border-bottom: 1px solid #ccc;">Table reading, metadata validation, shared numerical helpers</td>
</tr>
<tr style="background: #fce4ec;">
  <td style="padding: 8px 16px; font-weight: bold;">Data layer</td>
  <td style="padding: 8px 16px;">Bundled actuarial table files (installed automatically with the package)</td>
</tr>
</table>
</div>

## Actuarial table types

Lactuca provides three table classes that mirror the standard actuarial taxonomy:

| Class | Decrement | Typical use |
|---|---|---|
| `LifeTable` | `qx` — annual mortality | Life insurance, pensions, annuities |
| `DisabilityTable` | `ix` — disability incidence | Income protection, long-term care (dependency) |
| `ExitTable` | `ox` — withdrawal / termination | Pension fund turnover, policy lapses |

All three are classified along two orthogonal dimensions (see {doc}`tables_taxonomy`):

- **Insured structure** — *Aggregate* tables apply uniform rates to all insured persons;
  *Select-Ultimate* tables distinguish recently underwritten lives from the general portfolio.
- **Temporal nature** — *Static* tables use a fixed rate schedule with no improvement projection;
  *Generational* tables project rates forward from a base year using improvement factors
  tied to the insured's birth cohort. Generational tables require the `cohort=` parameter.

See {doc}`bundled_tables` for every bundled table identifier.

## Loading a table

Before performing any calculation, create a table object. All constructors share the
same two required arguments: the table identifier (a string) and the sex code
(`"m"` for male, `"f"` for female).

### Static life table

Static tables need no cohort. The object is ready to use immediately:

```python
from lactuca import LifeTable

# PASEM 2020, 1st order — Spanish annuity reserves (conservative)
pasem = LifeTable("PASEM2020_Rel_1o", "m")   # male
pasem_f = LifeTable("PASEM2020_Rel_1o", "f") # female

print(f"pasem.sex: {pasem.sex}, pasem_f.sex: {pasem_f.sex}")
```

### Generational life table (cohort required)

For generational tables, pass the insured's birth year via `cohort=`. The improvement
factors are applied automatically:

```python
from lactuca import LifeTable

# PER 2020 individual, 1st order — Spanish pension annuitants (born 1969)
per = LifeTable("PER2020_Ind_1o", "m", cohort=1969)

# DAV 2004 R aggregate, 2nd order — German private annuitants (born 1960)
dav = LifeTable("DAV2004R_Agg_2o", "m", cohort=1960)

# GAM 1994 with Scale AA — US group annuity with projected improvement (born 1955)
gam = LifeTable("GAM94_AA", "m", cohort=1955)

print(f"per.cohort: {per.cohort}, dav.cohort: {dav.cohort}, gam.cohort: {gam.cohort}")
```

### Select-ultimate table

Select-Ultimate tables distinguish recently underwritten lives (select period) from the
general portfolio (ultimate period). Pass `duration=` to specify years-since-underwriting;
`"ult"` explicitly selects the ultimate column:

```python
from lactuca import LifeTable

# DAV 2004 R Select-Ultimate (generational) — 2 years since underwriting
dav_su = LifeTable("DAV2004R_SelUlt_1o", "m", cohort=1960, duration=2)

# Same table — ultimate column (select period exhausted)
dav_ult = LifeTable("DAV2004R_SelUlt_1o", "m", cohort=1960, duration="ult")

print(f"dav_su.duration: {dav_su.duration}, dav_ult.duration: {dav_ult.duration}")
```

Some tables follow a **Duration-0** convention where the first select year is indexed as
`duration=0` rather than `duration=1`.  The AM92/AF92 series (UK) is one such table.
Check `table.start_duration` to see the minimum valid duration for any table:

```python
from lactuca import LifeTable

# AM92/AF92 — UK select-ultimate (static), Duration-0 convention
am92_d0  = LifeTable("AM92_AF92", "m", duration=0)     # freshly underwritten
am92_d1  = LifeTable("AM92_AF92", "m", duration=1)     # 1 year since underwriting
am92_ult = LifeTable("AM92_AF92", "m", duration="ult") # ultimate column

print(f"start_duration: {am92_d0.start_duration}")  # 0
print(f"select_period:  {am92_d0.select_period}")   # 2
print(f"qx(17, d=0): {am92_d0.qx(17):.6f}")        # 0.000427
```

The `duration=` parameter is supported by all three table classes.
See {doc}`tables_taxonomy` for the full Select-Ultimate type classification.

### Disability table

`DisabilityTable` models the annual probability of transitioning from an active (healthy)
state into disability. It is the standard tool for income-protection and long-term care
products. The constructor signature is identical to `LifeTable`; the only difference is
that the file must contain disability-incidence columns (`ix_m`, `ix_f`, or `ix_u`):
```python
from lactuca import DisabilityTable

# PEAI 2007 IAP individual — Spanish disability incidence (ages 18–64, period)
disability = DisabilityTable("PEAI2007_IAP_Ind", "m")

print(f"Disability ix at age 18: {disability.ix(18):.8f}")
```

### Exit (withdrawal) table

`ExitTable` models the annual probability of voluntarily leaving the insured portfolio
— policy surrender, employment termination, or any other cause of exit that is neither
death nor disability. The constructor signature again mirrors `LifeTable`; the table
file must contain exit-rate columns (`ox_m`, `ox_f`, or `ox_u`):
```python
from lactuca import ExitTable

exit_tbl = ExitTable("DummyEXIT", "m")

print(f"Exit ox at age 18: {exit_tbl.ox(18):.4f}")
```

### Unisex blend

All three table classes (`LifeTable`, `DisabilityTable`, `ExitTable`) support `sex="u"`
with a `unisex_blend` weight in `[0.0, 1.0]` to blend male and female decrement rates.
The formula applied is $q_u = w \cdot q_m + (1 - w) \cdot q_f$, where $w$ = `unisex_blend`
(0.0 = all female, 0.5 = equal blend, 1.0 = all male):

```python
from lactuca import LifeTable, DisabilityTable, ExitTable

# Life table — 60 % male / 40 % female blend (PASEM 2020)
pasem_u = LifeTable("PASEM2020_Rel_1o", "u", unisex_blend=0.6)

# Disability table — 50/50 blend
disability_u = DisabilityTable("PEAI2007_IAP_Ind", "u", unisex_blend=0.5)

# Exit table — 40 % male / 60 % female blend
exit_u = ExitTable("DummyEXIT", "u", unisex_blend=0.4)

print(f"pasem_u.unisex_blend: {pasem_u.unisex_blend}")
print(f"disability_u.unisex_blend: {disability_u.unisex_blend}")
print(f"exit_u.unisex_blend: {exit_u.unisex_blend}")
```

:::{note}
`unisex_blend` requires the table to carry **both** male and female rates (`qx_m` and `qx_f`).
Tables that include only one sex — for example the Chilean CMF tables (`CB_H_2020` ♂,
`RV_M_2020` ♀) or test tables like `DummyLIFE_1Sm` — cannot be blended and will raise a
`ValueError`.
:::

### Inspect a loaded table

Once a table is loaded, its key attributes are accessible as properties. You can also
create both male and female instances in a single call by passing a tuple of sex codes
as the second argument — each instance is fully independent:

```python
from lactuca import LifeTable

ltm, ltf = LifeTable("PASEM2020_Rel_1o", ("m", "f"))  # returns two independent instances

print(ltm.summary())     # formatted overview of the table
print(ltm.table_name)    # "PASEM2020_Rel_1o"
print(ltm.table_type)    # "life"
print(ltm.omega)         # terminal age ω
print(f"ltm.sex = {ltm.sex}, ltf.sex = {ltf.sex}")
```

### Accessing raw table data with TableSource

Every table instance exposes the parsed `.ltk` file via the `.table` property,
which returns a ``TableSource`` object with metadata and the raw decrement arrays:

```python
from lactuca import LifeTable

pasem = LifeTable("PASEM2020_Rel_1o", "m")

ts = pasem.table                  # TableSource instance
print(ts.table_name)              # "PASEM2020_Rel_1o"
print(ts.valid_sexes)             # ['f', 'm']
print(ts.data.head())             # first rows of the Polars DataFrame
```

You can also load a ``TableSource`` directly when you only need to inspect a file:

```python
from lactuca import TableSource

ts = TableSource("PASEM2020_Rel_1o")   # .ltk extension optional
print(ts)                              # formatted summary of all metadata
```

For the full attribute reference see {class}`lactuca.TableSource`.

### Default interest rate

`LifeTable` carries an optional `interest_rate` attribute
that serves as the default for all annuity, insurance, and endowment calculations.
It can be set at construction time or assigned later:

```python
from lactuca import LifeTable, InterestRate

# At construction time
pasem_3pct = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)
print(f"pasem_3pct.interest_rate = {pasem_3pct.interest_rate}")

pasem = LifeTable("PASEM2020_Rel_1o", "m")
print(f"pasem.interest_rate = {pasem.interest_rate}")

# Or assign afterwards
pasem.interest_rate = 0.03
print(f"pasem.interest_rate = {pasem.interest_rate}")

# Variable rate curve as default
pasem.interest_rate = InterestRate(terms=[5, 5], rates=[0.02, 0.025, 0.035])
print(f"pasem.interest_rate = {pasem.interest_rate}")
```

Passing `ir=` explicitly in any method call overrides the table default for that call
only; `interest_rate` is left unchanged:

```python
from lactuca import LifeTable

pasem = LifeTable("PASEM2020_Rel_1o", "m")
pasem.interest_rate = 0.03
a_default  = pasem.äx(65)           # uses 0.03
a_override = pasem.äx(65, ir=0.05)  # uses 0.05; interest_rate still 0.03

print(f"a_default  = {a_default:.6f}")
print(f"a_override = {a_override:.6f}")
```

When creating multiple instances at once with vectorial syntax, `interest_rate=` is
assigned to every instance in the same call — see {ref}`vectorial-interest-rate`.

## Survival and mortality probabilities

The simplest call returns a single probability for a single age. The same methods also
accept multiple ages at once or no argument at all to return the complete vector for
every age in the table.

### Single integer age

```python
from lactuca import LifeTable

lt = LifeTable("PASEM2010", "u", unisex_blend=0.45)

# Annual survival probability:  ₁p₅₀
p50 = lt.px(50)
print(f"p50 = {p50:.6f}")

# Annual mortality probability:  q₅₀  (= 1 − p₅₀ for life tables)
q50 = lt.qx(50)
print(f"q50 = {q50:.6f}")
```

### All ages — complete table vector

Call without arguments to retrieve probabilities for every age from 0 through ω.
The result is a NumPy array indexed by age:

```python
from lactuca import LifeTable
import matplotlib.pyplot as plt

lt = LifeTable("PASEM2010", "u", unisex_blend=0.45)

all_px = lt.px()  # NDArray, length ω + 1;  all_px[50] == p50
all_qx = lt.qx()  # NDArray, same length

# Example: plot qx on a log scale
plt.semilogy(all_qx, label="PASEM 2010 Unisex")
plt.xlabel("Age")
plt.ylabel("$q_x$")
plt.legend()
plt.show()
```

### Multiple ages at once

Pass a list or NumPy array to compute probabilities for several ages simultaneously.
The return value is a 1-D NumPy array in the same order as the input:

```python
from lactuca import LifeTable

lt = LifeTable("PASEM2010", "f")

ages = [40, 50, 60, 65, 70]
px_arr = lt.px(ages)    # NumPy array of 5 survival probabilities
qx_arr = lt.qx(ages)    # NumPy array of 5 mortality probabilities
print(px_arr)
print(qx_arr)
```

### Survivors and deaths: lx and dx

Two fundamental table quantities complement the probability methods: $\ell_x$ (number
of survivors at exact age $x$ from a radix cohort) and $d_x = l_x - l_{x+1}$
(deaths between consecutive integer ages). Fractional ages are supported via $l_x$
interpolation. For the full definitions — including the mid-year lives $L_x$ and its
role in life expectancy — see {doc}`../formulas`.

```python
from lactuca import LifeTable

pasem = LifeTable("PASEM2020_Rel_1o", "m")

# Survivors and deaths at integer ages
lx_65 = pasem.lx(65)
dx_65 = pasem.dx(65)

# Complete vectors for all ages (integers 0 … ω)
all_lx = pasem.lx()    # NDArray, length ω + 1
all_dx = pasem.dx()    # NDArray, length ω + 1

# Multiple ages at once
lx_arr = pasem.lx([50, 55, 60, 65])
dx_arr = pasem.dx([50, 55, 60, 65])
```

### Fractional ages

All probability methods accept non-integer ages. Lactuca interpolates $\ell_x$ between
integer ages using the method set in `config.lx_interpolation`: `"linear"` (equivalent
to UDD; default) or `"exponential"` (constant force of mortality). See
{doc}`lx_interpolation` for the full derivation and comparison:

```python
from lactuca import LifeTable

pasem = LifeTable("PASEM2020_Rel_1o", "m")

# Survival from exact age 50.5 to exact age 51.5
p50_half = pasem.px(50.5)

# Quarter-year grid
frac_ages = [50.0, 50.25, 50.50, 50.75]
p50_quarters = pasem.px(frac_ages)    # NumPy array of 4 values
```

### Sub-annual frequency

Use the `m` parameter to obtain the per-period probability for payment frequencies of
2, 4, 12, 52 or 365 periods per year:

```python
from lactuca import LifeTable

pasem = LifeTable("PASEM2020_Rel_1o", "m")

# Monthly survival probability at age 50:   (1/12)p₅₀
p50_monthly = pasem.px(50, m=12)

# Quarterly mortality probability
q50_quarterly = pasem.qx(50, m=4)
```

### Multi-period survival and mortality

`tpx` and `tqx` compute ${}_{t}p_x$ and ${}_{t}q_x$ for an arbitrary duration `t`
(keyword-only argument). With `t=1` they are equivalent to `px` and `qx`:

```python
from lactuca import LifeTable

pasem = LifeTable("PASEM2020_Rel_1o", "m")

# 10-year survival:   ₁₀p₅₀
ten_px = pasem.tpx(50, t=10)

# 5-year mortality:   ₅q₄₀
five_qx = pasem.tqx(40, t=5)

# Fractional t — 18-month survival from age 50
p50_18m = pasem.tpx(50, t=1.5)
```

Pass an array to `t` to compute a whole survival curve in a single call:

```python
from lactuca import LifeTable
import numpy as np

pasem = LifeTable("PASEM2020_Rel_1o", "m")

t_values = np.arange(1, 21)                     # t = 1, 2, ..., 20
survival_curve = pasem.tpx(50, t=t_values)       # 1-D array, length 20
```

Pass arrays to both `x` and `t` to get a 2-D matrix (rows = starting ages,
columns = durations):

```python
from lactuca import LifeTable
import numpy as np

pasem = LifeTable("PASEM2020_Rel_1o", "m")

ages = np.array([50, 55, 60, 65])
durations = np.array([5, 10, 15, 20])
tpx_matrix = pasem.tpx(ages, t=durations)        # shape (4, 4)
```

## Life expectancy

`ex()` computes the **complete (entire) expectation of life** $\mathring{e}_x$ at integer
ages via the discrete approximation $\mathring{e}_x = T_x / l_x$, where
$T_x = \sum_{k=x}^{\omega-1} L_k$ and $L_k \approx (l_k + l_{k+1})/2$ under UDD.
`ex_continuous()` evaluates the same formula at fractional starting ages using the
trapezoidal rule with `m` sub-intervals per year (default `m=12`).
For the complete mathematical derivation see {doc}`../formulas`:

```python
from lactuca import LifeTable

pasem = LifeTable("PASEM2020_Rel_1o", "m")

# Complete life expectancy at integer age 65: ẽ₆₅
e_65 = pasem.ex(65)
print(f"Complete life expectancy at 65: {e_65:.2f} years")

# Complete life expectancy at a fractional age (ex_continuous requires non-integer ages)
e_65_cont = pasem.ex_continuous(65.5, m=12)
print(f"Complete life expectancy at 65.5: {e_65_cont:.2f} years")
```

## Commutation functions

Commutation functions — $D_x$, $N_x$, $C_x$, $M_x$ and their higher-order sums $S_x$,
$R_x$ — absorb discount factors and survival probabilities into reusable, age-indexed
building blocks. They enable classical closed-form expressions such as
$\ddot{a}_x = N_x / D_x$ and $A_x = M_x / D_x$, and are available as standalone
methods for custom calculations and for verifying results against classical formulas.
Note that Lactuca's annuity and insurance methods (`äx`, `ax`, `Ax`, etc.) do **not**
use commutation functions internally — all calculation modes, including
`discrete_precision`, always evaluate **exact payment-grid summation** over survival
probabilities and discount factors. For the full dependency chain and usage, see
{doc}`commutation_functions` and {doc}`calculation_modes`.

```python
from lactuca import LifeTable

ir = 0.03
pasem = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=ir)

# Dx, Nx, Cx, Mx at age 50 with 3% annual effective interest

Dx_val = pasem.Dx(50)
Nx_val = pasem.Nx(50)
Cx_val = pasem.Cx(50)
Mx_val = pasem.Mx(50)
print(f"Dx(50) = {Dx_val:.4f}")
print(f"Nx(50) = {Nx_val:.4f}")
print(f"Cx(50) = {Cx_val:.6f}")
print(f"Mx(50) = {Mx_val:.4f}")
```

## Interest rates

`InterestRate` encapsulates an effective annual rate used for discounting and
accumulation. It supports constant rates, piecewise term structures (yield curves),
and named scenario collections. Any `LifeTable` method accepts either an
`InterestRate` object or a plain `float` in the `ir=` argument; a plain `float` is
wrapped automatically. For the complete API — discount factors, accumulation,
nominal-rate conversions, and scenarios — see {doc}`interest_rates_guide`.

### Constant rate

```python
from lactuca import InterestRate

# 3% annual effective rate
ir = InterestRate(0.03)
```

### Piecewise term structure

```python
from lactuca import InterestRate

# 5 years at 2%, next 5 years at 2.5%, then 3.5% thereafter
ir_term = InterestRate(terms=[5, 5], rates=[0.02, 0.025, 0.035])

# Discount factor for t = 7 years
v7 = ir_term.vn(7)
print(f"v(7) = {v7:.6f}")
```

## Growth rates

`GrowthRate` models a per-period revaluation factor applied to benefit amounts in
annuities and insurances. It is not limited to escalating benefits: common
applications include pension CPI indexation, salary-scale projections, guaranteed
annual increases, and any context where the benefit amount changes from one
anniversary to the next. Like `InterestRate`, it supports constant rates, piecewise
schedules, geometric or arithmetic growth, and multi-scenario containers. For the
complete API, see {doc}`growth_rates_guide`.

### Constant growth

```python
from lactuca import GrowthRate

# Geometric (compound) growth: F(t) = (1+g)^t
gr_geom = GrowthRate(0.02)
print(f"geometric factor(3) = {gr_geom.factor(3):.4f}")   # (1.02)^3 ≈ 1.0612

# Arithmetic (linear additive) growth: F(t) = 1 + g·t
gr_arith = GrowthRate(0.02, growth_type='a')
print(f"arithmetic factor(3) = {gr_arith.factor(3):.4f}") # 1 + 0.02 × 3 = 1.06
```

### Piecewise schedule

```python
from lactuca import GrowthRate

# 1% for the first year, 2% thereafter (e.g. CPI step schedule)
gr = GrowthRate(rates=[0.01, 0.02], terms=[1])

print(f"factor(1) = {gr.factor(1):.6f}")   # 1.01     — anniversary 1
print(f"factor(2) = {gr.factor(2):.6f}")   # 1.0302   — 1.01 × 1.02
print(f"factor(3) = {gr.factor(3):.6f}")   # 1.050804 — 1.01 × 1.02²
```

## Configuration

Lactuca exposes a global singleton `config` that controls all calculations in the
session: calculation mode (discrete vs. continuous), interpolation method for
fractional ages, numerical rounding precision per output type, and the file-system
path for custom table files. Changes take effect immediately and persist until
explicitly reset or the process restarts.

```python
from lactuca import config

# Calculation mode
# Options: 'discrete_precision', 'discrete_simplified',
#          'continuous_precision', 'continuous_simplified'
config.calculation_mode = "discrete_precision"

# Interpolation method for fractional ages: 'linear' or 'exponential'
config.lx_interpolation = "linear"

# Rounding precision for individual output types
config.decimals.qx = 6            # mortality probability
config.decimals.Dx = 4            # commutation function Dx
config.decimals.annuities = 8     # annuity present values

# Days per year convention for date calculations
config.days_per_year = 365.25

# Path to custom .ltk table files (defaults to the included tables directory)
config.tables_path = "/path/to/my/custom/tables"
```

For the full list of settings, TOML-file format, and the persistence API, see
{doc}`configuration`.

## Life annuities

A life annuity pays a recurring benefit while the annuitant survives. Lactuca
provides four standard variants — whole life, temporary, deferred, and continuous
— at any payment frequency $m$ and with optional benefit escalation (`gr=`). When
`interest_rate` is set on the table, the `ir=` argument can be omitted.

### Whole life annuities

```python
from lactuca import LifeTable

lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)

# Whole life annuity-due (annual payments): äx(65)
adue = lt.äx(65)

# Monthly payments: äx^(12)(65)
amonthly = lt.äx(65, m=12)

# Annuity-immediate (payment at end of period): ax(65)
aimmediate = lt.ax(65)

# Geometrically escalating (2% p.a.): plain float is auto-wrapped as GrowthRate(0.02)
agrowth = lt.äx(65, gr=0.02)
```

### Temporary annuities

```python
from lactuca import LifeTable, GrowthRate

lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)

# 20-year temporary annuity-due: ä65:20|
atemp = lt.äx(65, n=20)

# 15-year temporary, monthly payments: ä50:15|^(12)
atemp_monthly = lt.äx(50, n=15, m=12)

# Simplest growth: plain float auto-wrapped as GrowthRate(0.02) — geometric compound
atemp_gr_float = lt.äx(65, n=20, gr=0.02)

# Equivalent explicit form: GrowthRate object (geometric, 2% p.a.)
atemp_geom = lt.äx(65, n=20, gr=GrowthRate(0.02))

# Arithmetic salary-scale (flat 2 pp additive per year)
atemp_arith = lt.äx(65, n=20, gr=GrowthRate(0.02, growth_type='a'))
```

### Deferred annuities

```python
from lactuca import LifeTable, config

config.decimals.annuities = 4
lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)

# 10-year deferred whole life annuity: 10|äx(55)
adef = lt.äx(55, d=10)
print(adef)         # → 11.3534

# 5-year deferred, 20-year temporary: 5|ä60:20|
adef_temp = lt.äx(60, n=20, d=5)
print(adef_temp)    # → 11.3491

config.reset()
```

For deferred insurances, fractional deferment, multi-frequency examples, and joint-life
deferred products, see {doc}`deferment`.

### Continuous annuities

Continuous mode is activated globally via `config.calculation_mode`:

```python
from lactuca import LifeTable, config

lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)
config.calculation_mode = "continuous_precision"

# Continuous whole life annuity: āx(65)
acont = lt.äx(65)

# Continuous temporary: ā65:20|
acont_temp = lt.äx(65, n=20)

config.calculation_mode = "discrete_precision"  # restore default
```

The default mode is `"discrete_precision"`. Changing it affects all subsequent calls
in the session until explicitly reset. For all available modes and their trade-offs,
see {doc}`calculation_modes` and {doc}`configuration`.

## Life insurances

### Whole life and term insurance

```python
from lactuca import LifeTable

lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)

# Discrete whole life insurance: Ax(50)
Ax_val = lt.Ax(50)

# 20-year term insurance: A¹x:20|(50)
Aterm = lt.Ax(50, n=20)

# Geometrically escalating term insurance (sum insured grows 2% per anniversary year)
Aterm_geom = lt.Ax(50, n=20, gr=0.02)
```

### Continuous insurance

```python
from lactuca import LifeTable, config

pasem = LifeTable("PASEM2020_Rel_1o", "m")
config.calculation_mode = "continuous_precision"

# Continuous whole life: Āx(50)
Ax_cont = pasem.Ax(50, ir=0.03)

config.calculation_mode = "discrete_precision"
```

### Pure endowment

```python
from lactuca import LifeTable

pasem = LifeTable("PASEM2020_Rel_1o", "m")

# 10-year pure endowment: 10Ex(40)
nEx_val = pasem.nEx(40, n=10, ir=0.03)
```

Joint-life and last-survivor insurances (e.g. `Axy`, `äxy`, `äjoint`) are covered in
{doc}`joint_life_calculations`.

## Disability tables

A disability (incidence) table models the annual rate at which active lives transition
into a disabled state. All constructor parameters — `sex=`, `cohort=`, `duration=`,
`unisex_blend=` — follow the same conventions as `LifeTable`.

`DisabilityTable` exposes the disability incidence probability via `ix()`, which
accepts the same argument forms as `qx()` on a life table:

```python
from lactuca import DisabilityTable

# PEAI 2007 IAP individual (Spanish disability table, period; ages 18–64)
dt = DisabilityTable("PEAI2007_IAP_Ind", "m")

# Annual disability incidence at age 40: ix(40)
ix_val = dt.ix(40)

# Multiple ages at once
ix_arr = dt.ix([40.1012, 45.25, 50, 55])

# Complete incidence vector for all ages
all_ix = dt.ix()

# Monthly disability incidence:  _(1/12)ix^(12)(40)
ix_monthly = dt.ix(40, m=12)
```

## Exit tables

Exit tables model the withdrawal or lapse rate — the probability that a policyholder
or pension-scheme member leaves the active portfolio for a reason other than death
(e.g., voluntary surrender or employment termination). Constructor parameters follow
the same conventions as `LifeTable` and `DisabilityTable`.

`ExitTable` exposes the withdrawal probability via `ox()`, with the same call patterns
as `qx()` and `ix()`:

```python
from lactuca import ExitTable

et = ExitTable("DummyEXIT", "m")

# Annual withdrawal probability at age 45
ox_val = et.ox(45)

# Multiple ages
ox_arr = et.ox([30.21, 40.30, 50])

# Complete withdrawal vector for all ages
all_ox = et.ox()
```

## Functional API

All calculation methods are also available as standalone module-level functions in
`lactuca.functional` — useful in pipelines and custom workflows. The functional
form takes the table instance as the first argument; all remaining arguments are
identical to the method call:

```python
from lactuca.functional import äx, ax, Ax, nEx, lx, qx, ix, ox
from lactuca import LifeTable, DisabilityTable, ExitTable

ir = 0.03

lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=ir)
dt = DisabilityTable("PEAI2007_IAP_Ind", "m")
et = ExitTable("DummyEXIT", "m")

äx(lt, 65)               # whole-life annuity-due
ax(lt, 65)               # whole-life immediate annuity
Ax(lt, 50)               # whole-life insurance
nEx(lt, 40, n=10)        # 10-year pure endowment

lx(lt, 65)               # survivors at age 65
qx(lt, 50)               # annual mortality probability at 50
ix(dt, 40)               # annual disability incidence at 40
ox(et, 45)               # annual withdrawal probability at 45
```

For the complete function reference, see {doc}`functional_api`.

## Time shift (`ts`) for off-anniversary reserves

The `ts` parameter displaces the start of an annuity or insurance calculation by
`ts` years from the policy-anniversary age. Its primary use is computing **reserves
at a valuation date other than an exact policy anniversary** — for example, when
3.25 years have elapsed since inception and the next anniversary falls in 0.75 years.

`ts` is conceptually different from the `t` argument of `tpx`/`tqx`: `t` is the
*duration* of a survival interval, whereas `ts` shifts the entire payment grid
forward so that all discount and survival factors are evaluated from age $x + ts$
onward.

```python
from lactuca import LifeTable

lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)

# Mid-year reserve: 15-year annuity-due at age 50, 6 months into the policy year
a_midyear = lt.äx(50, n=15, ts=0.5)

# Off-anniversary reserve: 3.25 years since inception (next anniversary in 0.75 yr)
a_offann = lt.äx(50, n=10, ts=0.25)
```

When `ts` is fractional **and a `GrowthRate` is active** (`gr=` specified), Lactuca
emits a `UserWarning` as a reminder that benefit revaluation follows whole-year
anniversaries, not fractional ones. Without a `GrowthRate`, fractional `ts` is
silently accepted. To reject fractional shifts entirely, set
`config.force_integer_ts = True`. Full details in {doc}`growth_conventions`.

## Next steps

- Explore the {doc}`../api/index` for detailed API documentation
- Check {doc}`../formulas` for mathematical foundations
- See {doc}`bundled_tables` for the complete list of bundled tables
- Review {doc}`../changelog` for version history

## Support

For bug reports, feature requests, and questions, see the {doc}`../support` page.

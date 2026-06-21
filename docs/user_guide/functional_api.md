# Functional API

Lactuca provides a functional API that mirrors the object-oriented interface. Every actuarial
calculation method available on `LifeTable`, `DisabilityTable`, and `ExitTable` has an
equivalent module-level function. All functional symbols are re-exported from the top-level
`lactuca` package —
`from lactuca import ax` is the canonical form; `from lactuca.functional import ax` is
equally valid. Both styles delegate to the same underlying implementation.

## OOP style vs functional style

```python
from lactuca import LifeTable, ax, config

lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)
config.decimals.annuities = 4

# OOP style
value = lt.ax(65, n=10)
print(value)   # 8.0767

# Functional style — identical result
value = ax(lt, 65, n=10)
print(value)   # 8.0767
```

The functional form takes the table as its first positional argument. All remaining
parameters are identical to the corresponding method signature.

## Common parameters

These parameters apply to both styles (`lt.ax(...)` and `ax(lt, ...)`).

### Default interest rate (`interest_rate=`)

Setting `interest_rate=` at construction establishes a default for every calculation on
that instance. The per-call `ir=` argument overrides it when needed:

```python
from lactuca import LifeTable, äx, Ax, config

lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)
config.decimals.annuities  = 4
config.decimals.insurances = 4

print(äx(lt, 65))              # 16.0899  (uses ir=0.03 from the table)
print(Ax(lt, 65, n=20))        # 0.2631   (same)
print(äx(lt, 65, ir=0.04))     # 14.5792  (explicit ir= overrides for this call only)
```

### Benefit growth (`gr=`)

Annuity and insurance functions accept `gr=` (a {class}`lactuca.GrowthRate` instance)
to price benefits that increase at a constant rate.
See {doc}`growth_rates_guide` for construction details and conventions.

### Irregular cash flows

For non-standard payment timing or variable benefit amounts, pass `cashflow_times=` and
`cashflow_amounts=` to the relevant function.
See {doc}`irregular_cashflows` for a complete guide.

## Controlling output precision

Output precision is governed globally by `config.decimals`. Setting a field once affects
all subsequent calls to that function family:

```python
from lactuca import config

config.decimals.annuities  = 4   # ax, äx, axy, ajoint, äjoint, …
config.decimals.insurances = 4   # Ax, Axy, Afirst, nEx, nExy, nEjoint, …
config.decimals.ex         = 4   # ex, ex_continuous
config.decimals.Lx         = 4   # Lx, Lx_continuous
config.decimals.Tx         = 2   # Tx, Tx_continuous
config.decimals.Dx         = 4   # Dx  (each commutation function has its own field:
config.decimals.Nx         = 4   #      also Nx, Sx, Cx, Mx, Rx)
```

See {doc}`decimals_rounding` for the full list of precision fields and their defaults.

---

## Probability functions

### Common to all table types (`lx`, `px`, `tpx`, `tqx`, `dx`)

These functions accept any table type — `LifeTable`, `DisabilityTable`, or `ExitTable`:

```python
from lactuca import LifeTable, lx, px, tpx, tqx, dx, config

lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)
config.decimals.lx  = 2
config.decimals.dx  = 2
config.decimals.px  = 6
config.decimals.tpx = 6
config.decimals.tqx = 6

print(lx(lt, 50))              # 979332.85
print(px(lt, 65))              # 0.992007
print(tpx(lt, 50, t=10))       # 0.965831
print(tqx(lt, 50, t=10))       # 0.034169
print(dx(lt, 50))              # 2158.74
```

With a `DisabilityTable` (`lx` models the active healthy lives):

```python
from lactuca import DisabilityTable, lx, tpx, config

dt = DisabilityTable("PEAI2007_IAP_Ind", "m")
config.decimals.lx  = 2
config.decimals.tpx = 6

print(lx(dt, 40))              # 994559.89
print(tpx(dt, 40, t=5))        # 0.995728
```

### `qx` — annual mortality (`LifeTable` only)

`qx` returns the annual probability of death. Calling it on a `DisabilityTable` or
`ExitTable` raises `NotImplementedError`:

```python
from lactuca import LifeTable, qx, config

lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)
config.decimals.qx = 6

print(qx(lt, 65))    # 0.007993
# qx(dt, 40)        # ← NotImplementedError: use ix() instead
```

### `ix` — disability incidence probability (`DisabilityTable` only)

```python
from lactuca import DisabilityTable, ix, config

dt = DisabilityTable("PEAI2007_IAP_Ind", "m")
config.decimals.ix = 6

print(ix(dt, 40))    # 0.000682
```

### `ox` — exit / turnover probability (`ExitTable` only)

```python
from lactuca import ExitTable, ox, config

et = ExitTable("DummyEXIT", "m")
config.decimals.ox = 6

print(ox(et, 40))    # 0.004000
```

## Life expectancy (`LifeTable` only)

`ex` computes the complete expectation of life $\mathring{e}_x$ at integer ages (discrete
UDD approximation). `ex_continuous` uses trapezoidal numerical integration and requires a
**fractional** (non-integer) starting age — see {doc}`numerical_precision` for details:

```python
from lactuca import LifeTable, ex, ex_continuous, config

lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)
config.decimals.ex = 4

print(ex(lt, 65))                        # 22.1308
print(ex_continuous(lt, 65.5))           # 21.7186
print(ex_continuous(lt, 65.5, m=52))     # 21.7186  (finer grid, same result)
```

## Person-years functions (`LifeTable` only)

`Lx` and `Tx` require integer ages. `Lx_continuous` and `Tx_continuous` require
**fractional** ages and use numerical integration (integer ages raise `ValueError`):

```python
from lactuca import LifeTable, Lx, Tx, Lx_continuous, Tx_continuous, config

lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)
config.decimals.Lx = 4
config.decimals.Tx = 2

print(Lx(lt, 65))                        # 912531.7226
print(Tx(lt, 65))                        # 20276056.41
print(Lx_continuous(lt, 50.5))           # 977145.7488
print(Tx_continuous(lt, 50.5))           # 34095551.11
```

## Life annuities (`LifeTable` only)

```python
from lactuca import LifeTable, ax, äx, config

lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)
config.decimals.annuities = 4

print(ax(lt, 65))                # 15.0899  (whole-life, postpayable)
print(äx(lt, 65, n=20, m=12))   # 13.2805  (temporary, prepayable, monthly)
print(äx(lt, 65, d=5, n=15))    # 8.9446   (deferred 5y, 15y term)
```

## Life insurances (`LifeTable` only)

```python
from lactuca import LifeTable, Ax, nEx, config

lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)
config.decimals.insurances = 4

print(Ax(lt, 65))           # 0.5393  (whole-life)
print(Ax(lt, 65, n=20))     # 0.2631  (term, 20 years)
print(nEx(lt, 65, n=10))    # 0.6583  (pure endowment, 10 years)
```

## Commutation functions (`LifeTable` only)

```python
from lactuca import LifeTable, Dx, Nx, Cx, Mx, config

lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)
config.decimals.Dx = 4
config.decimals.Nx = 4
config.decimals.Cx = 6
config.decimals.Mx = 4

print(Dx(lt, 65))     # 134142.8681
print(Nx(lt, 65))     # 2158347.9610
print(Cx(lt, 65))     # 1056.526809
print(Mx(lt, 65))     # 72339.6391
```

## Multi-life functions (`LifeTable` only)

:::{note}
All multi-life functions follow the same ordering convention: **the principal life
occupies index 0** in both the `tables` list and the `ages` list; remaining lives follow
in the same order.
:::

### Two-life joint-life annuities (`axy`, `äxy`)

Paid while **both lives are simultaneously alive** (payments stop at the first death):

```python
from lactuca import LifeTable, axy, äxy, config

ltx, lty = LifeTable("PASEM2020_Rel_1o", ["m", "f"], interest_rate=0.03)
config.decimals.annuities = 4

print(axy([ltx, lty], ages=[65, 62]))    # 13.7589
print(äxy([ltx, lty], ages=[65, 62]))    # 14.7589
```

### Generic n-life joint-life annuities (`ajoint`, `äjoint`)

`ajoint` and `äjoint` generalise to any number of lives. When called with two tables
they produce the same result as `axy` / `äxy`:

```python
from lactuca import LifeTable, ajoint, äjoint, config

ltx, lty = LifeTable("PASEM2020_Rel_1o", ["m", "f"], interest_rate=0.03)
config.decimals.annuities = 4

print(ajoint([ltx, lty], ages=[65, 62], n=20))   # 12.3405
print(äjoint([ltx, lty], ages=[65, 62], n=20))   # 13.0518
```

### Three-life joint-life annuities (`axyz`, `äxyz`)

Payments continue while **all three lives are simultaneously alive**:

```python
from lactuca import LifeTable, axyz, äxyz, config

ltx, lty = LifeTable("PASEM2020_Rel_1o", ["m", "f"], interest_rate=0.03)
ltz      = LifeTable("PASEM2020_Rel_1o", "f", interest_rate=0.03)
config.decimals.annuities = 4

print(axyz([ltx, lty, ltz], ages=[65, 62, 60]))   # 12.9700
print(äxyz([ltx, lty, ltz], ages=[65, 62, 60]))   # 13.9700
```

### Two-life first-death insurance (`Axy`)

`Axy` pays on the **first death** among the two lives:

```python
from lactuca import LifeTable, Axy, config

ltx, lty = LifeTable("PASEM2020_Rel_1o", ["m", "f"], interest_rate=0.03)
config.decimals.insurances = 4

print(Axy([ltx, lty], ages=[65, 62]))   # 0.5786
```

### Three-life first-death insurance (`Axyz`)

```python
from lactuca import LifeTable, Axyz, config

ltx, lty = LifeTable("PASEM2020_Rel_1o", ["m", "f"], interest_rate=0.03)
ltz      = LifeTable("PASEM2020_Rel_1o", "f", interest_rate=0.03)
config.decimals.insurances = 4

print(Axyz([ltx, lty, ltz], ages=[65, 62, 60]))   # 0.6019
```

### Generic n-life first-death insurance (`Afirst`)

```python
from lactuca import LifeTable, Afirst, config

ltx, lty = LifeTable("PASEM2020_Rel_1o", ["m", "f"], interest_rate=0.03)
config.decimals.insurances = 4

print(Afirst([ltx, lty], ages=[65, 62]))   # 0.5786
```

### Two-life joint pure endowment (`nExy`)

`nExy` pays 1 if **both** lives survive $n$ years:

```python
from lactuca import LifeTable, nExy, config

ltx, lty = LifeTable("PASEM2020_Rel_1o", ["m", "f"], interest_rate=0.03)
config.decimals.insurances = 4

print(nExy([ltx, lty], ages=[65, 62], n=10))   # 0.6308
```

### Three-life joint pure endowment (`nExyz`)

```python
from lactuca import LifeTable, nExyz, config

ltx, lty = LifeTable("PASEM2020_Rel_1o", ["m", "f"], interest_rate=0.03)
ltz      = LifeTable("PASEM2020_Rel_1o", "f", interest_rate=0.03)
config.decimals.insurances = 4

print(nExyz([ltx, lty, ltz], ages=[65, 62, 60], n=10))   # 0.6088
```

### Generic n-life joint pure endowment (`nEjoint`)

```python
from lactuca import LifeTable, nEjoint, config

ltx, lty = LifeTable("PASEM2020_Rel_1o", ["m", "f"], interest_rate=0.03)
config.decimals.insurances = 4

print(nEjoint([ltx, lty], ages=[65, 62], n=10))   # 0.6308
```

## Inspecting payment flows (`return_flows`)

All annuity and insurance functions accept `return_flows=True` to return a per-period
breakdown instead of a scalar present value.
See {doc}`inspecting_cashflows` for the full key reference and worked examples.

In batch mode (array `x`), `return_flows=True` aggregates expected cash flows across all
policies.  Pass `benefits=sums_insured` (shape `(N,)`) to weight each policy's contribution
by its sum insured or pension amount — the returned `total_pv` is then the portfolio BEL or
PVDBO directly when `return_flows=True` is used in a precision mode.  With
`return_flows=False`, `benefits=` remains available in all modes as a per-policy scaling
factor.

---

## Batch with multiple tables

When policies belong to **different tables** — mixed-sex portfolio, multiple cohorts, or
different select durations — pass a **list of `LifeTable` instances** as the first
argument.  The list must have one entry per policy; ages and other per-policy parameters
are arrays of the same length.

```python
from lactuca import LifeTable, äx, Ax, config

lt_m = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)
lt_f = LifeTable("PASEM2020_Rel_1o", "f", interest_rate=0.03)

config.decimals.annuities  = 4
config.decimals.insurances = 4

ages   = [60, 62, 65, 68]
tables = [lt_m, lt_f, lt_m, lt_f]   # one per policy

annuities  = äx(tables, ages, n=20)   # NDArray shape (4,)
insurances = Ax(tables, ages, n=20)   # NDArray shape (4,)
premiums   = insurances / annuities   # element-wise
```

For **two-life joint** batch calculations pass arrays for both lives via `ages=(x_arr, y_arr)`:

```python
from lactuca import äxy

x_ages = [60, 62, 65]
y_ages = [55, 58, 61]

# [lt_m, lt_f] — one table per life; arrays for batch over policies
result = äxy([lt_m, lt_f], ages=(x_ages, y_ages), n=20)
```

See {doc}`batch_calculations` for a complete guide: per-policy parameters, joint-life
batch, `return_flows=True` for BEL calculations, and performance notes.

## Full function list

| Function | Equivalent method | Applicable to |
|----------|------------------|---------------|
| `lx` | `table.lx(x)` | All |
| `dx` | `table.dx(x)` | All |
| `px` | `table.px(x, m)` | All |
| `tpx` | `table.tpx(x, t=)` | All |
| `tqx` | `table.tqx(x, t=)` | All |
| `qx` | `table.qx(x, m)` | `LifeTable` only |
| `ix` | `table.ix(x, m)` | `DisabilityTable` only |
| `ox` | `table.ox(x, m)` | `ExitTable` only |
| `ex` | `table.ex(x)` | `LifeTable` |
| `ex_continuous` | `table.ex_continuous(x, m=)` | `LifeTable` |
| `Lx`, `Tx` | `table.Lx(x)`, `table.Tx(x)` | `LifeTable` |
| `Lx_continuous`, `Tx_continuous` | `table.Lx_continuous(x, m=)`, `table.Tx_continuous(x, m=)` | `LifeTable` |
| `Dx`, `Nx`, `Sx` | `table.Dx(x, ir=)`, … | `LifeTable` |
| `Cx`, `Mx`, `Rx` | `table.Cx(x, ir=)`, … | `LifeTable` |
| `ax` | `table.ax(x, …)` | `LifeTable` |
| `äx` | `table.äx(x, …)` | `LifeTable` |
| `Ax` | `table.Ax(x, …)` | `LifeTable` |
| `nEx` | `table.nEx(x, …)` | `LifeTable` |
| `axy`, `äxy` | `table.axy(…)`, `table.äxy(…)` | `LifeTable` |
| `axyz`, `äxyz` | `table.axyz(…)`, `table.äxyz(…)` | `LifeTable` |
| `ajoint`, `äjoint` | `table.ajoint(…)`, `table.äjoint(…)` | `LifeTable` |
| `Axy` | `table.Axy(…)` | `LifeTable` |
| `Axyz` | `table.Axyz(…)` | `LifeTable` |
| `Afirst` | `table.Afirst(…)` | `LifeTable` |
| `nExy` | `table.nExy(…)` | `LifeTable` |
| `nExyz` | `table.nExyz(…)` | `LifeTable` |
| `nEjoint` | `table.nEjoint(…)` | `LifeTable` |

**DataFrame workflow:** all per-policy parameters (`x`/`ages`, `n`, `ir`, `d`, `ts`, `m`, `gr`,
`benefits`) accept Pandas and Polars `Series` directly.  A numeric `ir`/`gr` Series is
converted to per-policy floats; an object-dtype `Series` of `InterestRate` or `GrowthRate`
instances (including piecewise curves) is treated like a list.  See
{doc}`batch_calculations` — *DataFrame workflow* for complete examples.

## Global calculation mode (`config.calculation_mode`)

All functional wrappers delegate to the same OOP engines.  The active calculation mode
(`discrete_precision`, `discrete_simplified`, `continuous_precision`, `continuous_simplified`)
comes from the process-global `Config` singleton — there is **no** per-call mode argument
on `ax(lt, …)`, `Ax(lt, …)`, etc.:

```python
from lactuca import LifeTable, ax, config

config.calculation_mode = "discrete_simplified"
lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)
print(ax(lt, 65, n=10))   # discrete_simplified via config
```

`InterestRate.a()` / `.ä()` follow the same global mode.  Change `config.calculation_mode`
(or persist settings via the user config file) before invoking functional wrappers.
See {doc}`calculation_modes` and {doc}`configuration`.

## See also

- {doc}`commutation_functions` — mathematical background for Dx, Nx, etc.
- {doc}`calculation_modes` — how discrete / continuous modes affect method dispatch
- {doc}`interest_rates_guide` — `InterestRate` usage
- {doc}`growth_rates_guide` — `GrowthRate` for benefit growth rates (`gr=` parameter)
- {doc}`decimals_rounding` — controlling output precision with `config.decimals`
- {doc}`irregular_cashflows` — non-standard payment timing and variable benefits
- {doc}`tables_taxonomy` — `LifeTable`, `DisabilityTable`, and `ExitTable` explained
- {doc}`inspecting_cashflows` — per-period flow breakdowns with `return_flows=True`

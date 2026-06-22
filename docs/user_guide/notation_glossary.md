# Notation and Glossary

This page is the authoritative notation reference for the Lactuca documentation and API.
It maps every symbol, method name, and parameter to its actuarial definition.
For derivations and worked formulas consult {doc}`../formulas`; for configuration options see {doc}`configuration`.

## Life table symbols

| Symbol | Lactuca name | Description |
|--------|-------------|-------------|
| $x$ | `x` | Exact age (years) |
| $\omega$ | `omega` | Limiting age (first age where $l_\omega = 0$) |
| $l_x$ | `lx` | Number of lives at age $x$ (from radix $l_0$) |
| $q_x$ | `qx` | Probability of death in $[x, x+1)$ |
| $p_x$ | `px` | Probability of survival in $[x, x+1)$; $p_x = 1 - q_x$ |
| $d_x$ | `dx` | Expected deaths in $[x, x+1)$; $d_x = l_x - l_{x+1}$ |
| $L_x$ | `Lx` | Person-years lived in $[x, x+1)$ |
| $T_x$ | `Tx` | Total person-years lived from age $x$ (integer ages) |
| $T_{x+s}$ | `Tx_continuous(x, m=)` | Total person-years lived from fractional age $x+s$, $0 < s < 1$; numerical integration with $m$ sub-intervals/year |
| $\mathring{e}_x$ | `ex` | Complete life expectancy at integer age $x$; $\mathring{e}_x = T_x / l_x$ |
| $\mathring{e}_{x+s}$ | `ex_continuous(x, m=)` | Complete life expectancy at fractional age $x+s$, $0 < s < 1$ (numerical integration) |
| ${}_t p_x$ | `tpx` | Probability of surviving $t$ years from age $x$ |
| ${}_t q_x$ | `tqx` | Probability of dying within $t$ years from age $x$ |

## Decrements

| Symbol | Lactuca column | Table class | Meaning |
|--------|---------------|-------------|---------|
| $q_x$ | `qx_m`, `qx_f`, `qx_u` | `LifeTable` | Annual mortality rate |
| $i_x$ | `ix_m`, `ix_f` | `DisabilityTable` | Annual disability incidence rate |
| $o_x$ | `ox_m`, `ox_f` | `ExitTable` | Annual exit/withdrawal rate |

## Interest rate symbols

| Symbol | Lactuca | Description |
|--------|---------|-------------|
| $i$ | `InterestRate(i)` or `table.interest_rate = i` | Annual effective interest rate |
| $v$ | `ir.vn(n)` computes $v^n = (1+i)^{-n}$ | Discount factor; $v = 1/(1+i)$ |
| $\delta$ | `ir.delta(t=)` | Force of interest; $\delta = \ln(1+i)$ for constant rates (`ir.delta()`). For piecewise term structures, provide `t` (`ir.delta(t)`) |
| $i^{(m)}$ | — (no direct method; used implicitly via parameter `m`) | Nominal rate convertible $m$ times per year |
| $d^{(m)}$ | — (no direct method; used implicitly via parameter `m`) | Nominal discount rate convertible $m$ times per year; $d^{(m)} = m\left(1 - v^{1/m}\right)$ |

## Multi-scenario rate containers

| Term | Lactuca meaning |
|------|-----------------|
| Active scenario | Name of the sub-curve (`base`, `stress`, …) used by `vn()`, `ax()`, `factor()`, etc. on **this** call |
| Reference semantics | `LifeTable.interest_rate` and `ir=` / `gr=` share the container object; the active scenario is **not** snapshotted at construction |
| `copy()` | Deep copy that isolates `active_scenario` and sub-curves from the parent container |

## Commutation functions

| Symbol | Lactuca method | Formula |
|--------|---------------|---------|
| $D_x$ | `Dx` | $v^x \cdot l_x$ |
| $N_x$ | `Nx` | $\sum_{k=x}^{\omega} D_k$ |
| $S_x$ | `Sx` | $\sum_{k=x}^{\omega} N_k$ |
| $C_x$ | `Cx` | $v^{x+1} \cdot d_x$ |
| $M_x$ | `Mx` | $\sum_{k=x}^{\omega} C_k$ |
| $R_x$ | `Rx` | $\sum_{k=x}^{\omega} M_k$ |
| $L_x$ | `Lx` | $\int_0^1 l_{x+t}\,dt \approx (l_x + l_{x+1})/2$ (integer ages) |
| $L_{x+s}$ | `Lx_continuous(x, m=)` | Person-years lived in $[x+s,\, x+s+1)$ at fractional age; numerical integration with $m$ sub-intervals/year |

:::{note}
$L_x$ and $T_x$ are life-table demographic quantities rather than discounted commutation functions,
but Lactuca exposes them alongside $D_x$, $N_x$, etc. as `LifeTable` methods.
Their continuous variants `Lx_continuous` and `Tx_continuous` support fractional starting ages
via numerical integration.
:::

## Life annuity symbols

| Symbol | Lactuca method | Description |
|--------|---------------|-------------|
| $a_x$ | `ax` | Whole-life immediate annuity |
| $\ddot{a}_x$ | `äx` | Whole-life annuity-due |
| $a_{x:\overline{n}\vert}$ | `ax(n=n)` | Temporary (term) immediate annuity |
| $\ddot{a}_{x:\overline{n}\vert}$ | `äx(n=n)` | Temporary annuity-due |
| ${}_{d\vert}\ddot{a}_{x:\overline{n}\vert}$ | `äx(d=d, n=n)` | Deferred temporary annuity-due |
| $\ddot{a}_{x:\overline{n}\vert}^{(m)}$ | `äx(m=m)` | Annuity-due with frequency $m$ |
| $a_{xy}$ | `axy` | Joint-life immediate annuity (two lives; pays while both survive) |
| $\ddot{a}_{xy}$ | `äxy` | Joint-life annuity-due (two lives; pays while both survive) |
| $a_{x_1 x_2 \cdots x_n}$ | `ajoint` | Joint-life immediate annuity ($n$ lives; pays while all survive) |
| $\ddot{a}_{x_1 x_2 \cdots x_n}$ | `äjoint` | Joint-life annuity-due ($n$ lives; pays while all survive) |

## Life insurance symbols

| Symbol | Lactuca method | Description |
|--------|---------------|-------------|
| $A_x$ | `Ax` | Whole-life insurance |
| $A_{x:\overline{n}\vert}^1$ | `Ax(n=n)` | Term insurance |
| ${}_n E_x$ | `nEx` | Pure endowment |
| $A_{xy}$ | `Axy` | First-death insurance (two lives) |
| ${}_n E_{xy}$ | `nExy` | Joint pure endowment (two lives; both must survive $n$ years) |
| $A_{xyz}$ | `Axyz` | First-death insurance (three lives) |
| ${}_n E_{xyz}$ | `nExyz` | Joint pure endowment (three lives; all must survive $n$ years) |
| $A_{x_1 \cdots x_n}$ | `Afirst` | First-death insurance ($n$ lives) |

## Calculation parameters

Most parameters are shared by annuity (`ax`, `äx`, `axy`, `ajoint`, …) and insurance
(`Ax`, `Axy`, `Afirst`, `nEx`) functions.  Annuity-only parameters are noted.

| Parameter | Type | Meaning |
|-----------|------|---------|
| `x` | `float` | Entry age in years |
| `n` | `float` or `None` | Term / duration in years; `None` = whole life |
| `m` | `int` | Payment frequency per year. Valid values: 1 (annual), 2 (semi-annual), 3 (every 4 months), 4 (quarterly), 6 (bi-monthly), 12 (monthly), 14 (Spanish "14 pagas", approximate), 24 (twice-monthly), 26 (biweekly), 52 (weekly), 365 (daily). See {doc}`calculation_modes` for details and caveats on `m=14`. |
| `d` | `float` | Deferment period in years |
| `ts` | `float` | Temporal shift from policy origin (years already elapsed) |
| `ir` | `float`, `InterestRate`, or `None` | Interest rate for discounting; if `None`, uses the table's `interest_rate` attribute |
| `gr` | `float`, `GrowthRate`, or `None` | Benefit growth rate per year |
| `cashflow_times` | `Sequence[float]` or `None` | Custom payment times in years since origin; overrides `m`. |
| `cashflow_amounts` | `Sequence[float]` or `None` | Custom benefit amount per payment; disables `gr`. |
| `return_flows` | `bool` | If `True`, return detailed cash-flow breakdown |

Payment timing is determined by the method name: `äx`/`äjoint` = annuity-due (prepayable); `ax`/`ajoint` = annuity-immediate (postpayable).

## Calculation modes

All four modes value the same actuarial present values under shared conventions; they are
**actuarially coherent** but not numerically identical. See {ref}`actuarial-coherence-of-modes`
in {doc}`calculation_modes`.

| Mode string | Type | Handling of $m > 1$ | Notes |
|-------------|------|---------------------|-------|
| `"discrete_precision"` | Discrete | Via $l_x$ interpolation | Default; production reference |
| `"discrete_simplified"` | Discrete | Woolhouse (annuities) / linear age interp. (insurances); hybrid m-thly tail when $n$ fractional | Faster approximations |
| `"continuous_precision"` | Continuous | Numerical integration | Unrounded $l_x$ integration |
| `"continuous_simplified"` | Continuous | Trapezoidal / averaging shortcuts | Fast continuous approximations |

See {doc}`calculation_modes` for details.

## Force of mortality methods

The `force_mortality_method` setting controls how $\mu_{x+t}$ is estimated from discrete
$({}_t p_x)$ values.  It is active only under `"continuous_precision"` and
`"continuous_simplified"` calculation modes.  Set via `config.force_mortality_method`.

| Value | Method | Best used for |
|-------|--------|--------------|
| `"finite_difference"` | Central finite differences of $\ln({}_t p_x)$ | Default; standard actuarial tables |
| `"spline"` | Cubic spline of $\ln({}_t p_x)$ with analytical derivative | Smooth graduated mortality curves |
| `"kernel"` | Gaussian kernel smoothing with numerical differentiation | Noisy or irregular empirical data |

## Other terms

### Life-table fundamentals

**Radix** ($l_0$)
: The initial population size from which the life table is constructed.  All subsequent $l_x$ values are
  proportional to $l_0$.  Lactuca default: $l_0 = 1\,000\,000$.

**Limiting age** ($\omega$)
: The highest age in the table for which $l_\omega = 0$ (or $q_{\omega-1} = 1$).

**Force of mortality** ($\mu_x$)
: The instantaneous hazard rate of death; $\mu_x = -\frac{\mathrm{d}}{\mathrm{d}x} \ln l_x$.
  Used in continuous calculations.  The numerical approximation method is controlled by
  `config.force_mortality_method`; see the **Force of mortality methods** section above.

### Table types and classification

**Period table**
: A mortality table whose $q_x$ values represent a single calendar year, with no cohort improvement
  applied.

**Generational table**
: A mortality table whose $q_x$ values are adjusted along the life-table diagonal using improvement
  factors for the specified birth cohort.

**Cohort**
: Birth year of the insured, used to project age-specific mortality along the life-table diagonal
  in generational tables.  Passed as `cohort=` to `LifeTable`.

**Mortality Improvement (MI)**
: The systematic reduction of mortality rates over time.  In Lactuca, MI parameters are stored
  in `mi_*` columns of `.ltk` files alongside the base rates.  The projection formula is
  specified by the table's `improvement_formula` metadata field:

  | Formula | MI identifier | Expression |
  |---------|--------------|------------|
  | `exponential_improvement` | $\lambda_x$ | $q_{x,t} = q_x^{(b)} \cdot e^{-\lambda_x(t-t_0)}$ |
  | `linear_improvement` | $\lambda_x$ | $q_{x,t} = q_x^{(b)} - \lambda_x(t-t_0)$ |
  | `discrete_improvement` | $AA_x$ | $q_{x,t} = q_x^{(b)} \cdot (1-AA_x)^{t-t_0}$ |
  | `projected_improvement` | $AA_{x,t}$ | $q_{x,t} = q_x^{(b)} \cdot \prod_{t'=t_0+1}^{c+x}(1-AA_{x,t'})$ |

  `mi_m`/`mi_f` columns hold constant-per-age factors ($\lambda_x$ or $AA_x$); `mi_m_YYYY`/`mi_f_YYYY`
  columns hold year-indexed factors ($AA_{x,t}$).  The only public input required is `cohort=` (birth
  year); Lactuca resolves the diagonal automatically.
  See {doc}`mortality_improvement` for the full reference.

**First-order table**
: A mortality table incorporating a prudential safety margin, used for pricing and regulatory
  reserving (Spanish: *primera orden*; German: *1. Ordnung*).  Denoted `_1o` in Lactuca table
  identifiers.

**Second-order table**
: A mortality table based on central best-estimate mortality rates, without prudential margins
  (Spanish: *segunda orden*; German: *2. Ordnung*).  Denoted `_2o` in Lactuca table identifiers.
  Typical uses: experience studies, stochastic projections, asset-liability modelling.

**Aggregate table**
: A mortality table in which $q_x$ depends only on attained age $x$, with no distinction by duration
  since underwriting.  Combines pre- and post-selection experience into a single rate per age.
  The complement of a select-ultimate table.

**Select-ultimate table**
: A mortality table in which mortality rates depend on both attained age $x$ and duration $d$ since
  underwriting for $d < $ select period, written $q_{[x]+d}$.  Once $d \ge$ select period, rates
  revert to the **ultimate** column $q_x^u$, which depends only on attained age.

**Select period**
: The number of years after underwriting during which mortality depends on both attained age and
  duration since selection.  After this period expires, the **ultimate** rates apply.
  Accessible via `LifeTable.select_period`.

**Duration** (select offset)
: Years elapsed since the underwriting date (integer $\ge 0$).  Determines which select column is
  used for select-ultimate tables.  Passed as `duration=` to `LifeTable`.

**Start duration**
: Minimum valid integer duration for a select-ultimate table.  Most tables use `start_duration=1`
  (columns named `qx_m_s1` …).  Tables following the CMI/UK convention — such as AM92/AF92 — use
  `start_duration=0` (columns named `qx_m_s0`, `qx_m_s1` …).  Accessible via
  `TableSource.start_duration` and `LifeTable.start_duration`.  Passing a duration below
  `start_duration` raises `ValueError`.

**Ultimate**
: The section of a select-ultimate table where $q_x$ depends only on attained age — i.e., after
  the select period has expired.  Equivalent to an aggregate table restricted to post-selection lives.

### Fractional-age assumptions

**UDD (Uniform Distribution of Deaths)**
: Fractional-age assumption that deaths are uniformly distributed within each year of age:
  $l_{x+s} = (1-s) l_x + s \cdot l_{x+1}$.  Set via `config.lx_interpolation = "linear"` (default).

**CFM (Constant Force of Mortality)**
: Fractional-age assumption that $\mu_x$ is constant within $[x, x+1)$:
  $l_{x+s} = l_x \cdot p_x^s$.  Set via `config.lx_interpolation = "exponential"`.

### Actuarial building blocks

**Commutation function**
: Precomputed discounted survival quantity ($D_x$, $N_x$, $C_x$, $M_x$, etc.) used to evaluate
  annuity and insurance values in closed form.  See {doc}`commutation_functions`.

**TableKey**
: A hashable `NamedTuple` (`lactuca.TableKey`) that uniquely identifies a `LifeTable`,
  `DisabilityTable`, or `ExitTable` instance by its five constructor dimensions:
  `table_name` (str), `sex` (str), `cohort` (int or None), `duration` (int/str or None),
  and `unisex_blend` (float or None).  `table_name` and `sex` are required; the remaining
  fields default to `None`.  Used as the key type in `dict` returns from the vectorial
  constructor when `return_dict=True`.

  ```python
  from lactuca import TableKey
  key = TableKey("PER2020_Ind_1o", "m", 1960)    # cohort=1960, duration=None, unisex_blend=None
  key = TableKey("PASEM2010", "u", unisex_blend=0.55)
  ```

  :::{note}
  Always look up a key using the **same literal float** that was passed as `unisex_blend` at
  construction time.  IEEE 754 rounding means `0.5 + 0.05 \neq 0.55` at the bit level.
  :::

# Actuarial Formulas

Mathematical foundations and formulas used throughout Lactuca.

This page is a concise mathematical reference for each actuarial quantity Lactuca
computes.  Sections are ordered from foundational concepts to derived products.  Each
section documents the formula in force, the required hypothesis or mode, and any
input constraints — including the constraints whose violation raises a `ValueError`.

For narrative explanations and runnable code examples see
{doc}`user_guide/life_annuities_guide`, {doc}`user_guide/life_insurances_guide`, and
{doc}`user_guide/calculation_modes`.  For a full list of errors see
{doc}`errors_reference`.

| Section | Key symbols |
|---|---|
| [Survival and mortality functions](#survival-and-mortality-functions) | $l_x$, $d_x$, ${}_tp_x$, ${}_tq_x$, $L_x$, $T_x$ |
| [Select-ultimate notation](#select-ultimate-notation) | $q_{[x]+d}$, select period |
| [Fractional age interpolation](#fractional-age-interpolation) | UDD (`"linear"`), CFM (`"exponential"`) |
| [Commutation functions](#commutation-functions) | $D_x$, $N_x$, $S_x$, $C_x$, $M_x$, $R_x$, $L_x$, $L_{x+s}$, $T_x$, $T_{x+s}$ |
| [Life annuities](#life-annuities) | $\ddot{a}_x$, temporary, deferred, $m$-thly, immediate, continuous |
| [Life insurances](#life-insurances) | $A_x$, term, endowment, pure endowment, deferred, continuous |
| [Annuity–insurance relationship](#relationship-between-annuities-and-insurances) | $\ddot{a}_x = (1 - A_x)/d$ |
| [Life expectancy](#life-expectancy) | $\mathring{e}_x$, $e_x$ |
| [Interest rate conversions](#interest-rate-conversions) | $i^{(m)}$, $\delta$, $d^{(m)}$ |
| [Growth rates (escalating benefits)](#growth-rates-escalating-benefits) | $v_g = (1+g)/(1+i)$ |
| [Multiple decrements](#multiple-decrements) | ${}_tq_x^{(j)}$, ${}_tp_x^{(\tau)}$ |
| [Joint life and last survivor](#joint-life-and-last-survivor) | $\ddot{a}_{xy}$, $\ddot{a}_{\overline{xy}}$, $A_{xy}$ |
| [Formulas by calculation mode](#formulas-by-calculation-mode) | mode-specific formulas |
| [References](#references) | — |

(formulas-survival)=
(survival-and-mortality-functions)=
## Survival and mortality functions

### Life table functions

The fundamental survival function $l_x$ represents the number of lives at exact age $x$ from an initial cohort $l_0$.

$$
l_x = l_0 \cdot \prod_{k=0}^{x-1} (1 - q_k)
$$

### Survival probability

The probability that a life aged $x$ survives to age $x+t$:

$$
{}_t p_x = \frac{l_{x+t}}{l_x}
$$

### Mortality probability

The probability that a life aged $x$ dies within $t$ years:

$$
{}_t q_x = 1 - {}_t p_x = \frac{l_x - l_{x+t}}{l_x}
$$

### Deferred mortality

The probability of surviving $u$ years and then dying within the next $t$ years:

$$
{}_{u|t} q_x = {}_u p_x \cdot {}_t q_{x+u}
$$

### Survivors and deaths

The number of survivors in a cohort at aged $x$ is the **life table function** $l_x$. Deaths
between consecutive integer ages are:

$$
d_x = l_x - l_{x+1}
$$

The **central exposed-to-risk** (person-years lived in $[x, x+1)$) under UDD is
$L_x = l_x - d_x/2$. See the [Lx commutation function](#lx-function-person-years-lived)
definition below.

(formulas-select-notation)=
(select-ultimate-notation)=
## Select-ultimate notation

For select-ultimate tables, mortality depends on both attained age $x$ and duration $d$
since underwriting.  The conventional actuarial notation for the rate at duration $d$ is
$q_{[x]+d}$, read as "the rate for a life selected at age $x$ and now $d$ years into the
select period".

In Lactuca the minimum valid integer duration is given by `TableSource.start_duration`:

- **`start_duration = 1`** — standard convention (e.g. DAV 2004 R, most international tables);
  `duration=1` is the freshly-underwritten rate $q_{[x]}$.
- **`start_duration = 0`** — CMI/UK convention (e.g. AM92/AF92);
  `duration=0` is the freshly-underwritten rate $q_{[x]+0}$.

Durations at or beyond `start_duration + select_period` fall back to the ultimate column
$q_{x+d}^{\text{ult}}$ regardless of the value passed.

$$
q_{[x]+d} = \begin{cases}
  q_{[x]+d}^{\text{select}} & \text{if } d < \text{start duration} + \text{select period} \\
  q_{x+d}^{\text{ult}} & \text{if } d \ge \text{start duration} + \text{select period}
\end{cases}
$$

:::{seealso}
For the fallback formula, code example, and `select_period` metadata field see
[Select-and-ultimate tables](#select-and-ultimate-tables) below.
:::

---

(formulas-interpolation)=
(fractional-age-interpolation)=
## Fractional age interpolation

Lactuca supports two interpolation methods for $l_{x+s}$ at fractional ages,
controlled by `config.lx_interpolation`.

:::{note}
`config.lx_interpolation = "linear"` (UDD, **default**) or `"exponential"` (CFM).
Passing any other string raises `ValueError`.
The mode affects all probability and annuity calculations at fractional ages.
:::

### UDD — Uniform Distribution of Deaths (`"linear"`)

For $0 \leq s < 1$:

$$
l_{x+s} = (1-s)\,l_x + s\,l_{x+1}
$$

$$
{}_s p_x = 1 - s\,q_x, \qquad {}_{1/m}p_x = 1 - \frac{q_x}{m}
$$

Force of mortality under UDD (increasing throughout each year of age):

$$
\mu_{x+s} = \frac{q_x}{1 - s\,q_x}
$$

### CFM — Constant Force of Mortality (`"exponential"`)

For $0 \leq s < 1$:

$$
l_{x+s} = l_x \cdot p_x^{\,s}
$$

$$
{}_s p_x = p_x^{\,s} = e^{-\mu_x s}, \qquad {}_{1/m}p_x = p_x^{\,1/m}
$$

Force of mortality (truly constant within $[x, x+1)$ under CFM):

$$
\mu_{x+s} = -\ln p_x
$$

(formulas-commutation)=
(commutation-functions)=
## Commutation functions

Commutation functions simplify actuarial calculations by precomputing discounted
survival values.

:::{note}
**`Dx`, `Nx`, `Cx`, `Mx`, `Sx`, `Rx`, `Lx`, `Tx` require integer ages.**
Passing a fractional age raises `ValueError`.
`Lx_continuous` and `Tx_continuous` are the counterparts for fractional ages;
they raise `ValueError` for integer ages.
:::

| Symbol | Formula | Actuarial use | Lactuca method |
|---|---|---|---|
| $D_x$ | $v^x l_x$ | Denominator in all commutation formulas | `lt.Dx(x, ir=)` |
| $N_x$ | $\sum_{k \ge x} D_k$ | $\ddot{a}_x = N_x/D_x$ | `lt.Nx(x, ir=)` |
| $S_x$ | $\sum_{k \ge x} N_k$ | Increasing annuities | `lt.Sx(x, ir=)` |
| $C_x$ | $v^{x+f} d_x$ \* | Building block for $M_x$ | `lt.Cx(x, ir=)` |
| $M_x$ | $\sum_{k \ge x} C_k$ | $A_x = M_x/D_x$ | `lt.Mx(x, ir=)` |
| $R_x$ | $\sum_{k \ge x} M_k$ | Increasing insurance | `lt.Rx(x, ir=)` |
| $L_x$ | $(l_x + l_{x+1})/2$ | Person-years in $[x, x+1)$ (integer $x$) | `lt.Lx(x)` |
| $L_{x+s}$ | $\int_{x+s}^{x+s+1} l_t\,\mathrm{d}t$ | Person-years at fractional age | `lt.Lx_continuous(x+s, m=)` |
| $T_x$ | $\sum_{k=x}^{\omega-1} L_k$ | $\mathring{e}_x = T_x / l_x$ (integer $x$) | `lt.Tx(x)` |
| $T_{x+s}$ | $\int_{x+s}^{\omega} l_t\,\mathrm{d}t$ | Future person-years at fractional age | `lt.Tx_continuous(x+s, m=)` |

\* $f$ = mortality placement fraction: `"mid"` → 0.5 (default); `"end"` → 1.0; `"beginning"` → 0.
See the [C function definition](#c-function-deaths) for details.

### Discount factor

$$
v = \frac{1}{1+i}
$$

### D function

$$
D_x = v^x \cdot l_x
$$

### N function (sum of D)

$$
N_x = \sum_{k=x}^{\omega} D_k = D_x + D_{x+1} + \cdots + D_{\omega}
$$

### S function (sum of N)

$$
S_x = \sum_{k=x}^{\omega} N_k
$$

(c-function-deaths)=
### C function (deaths)

$$
C_x = v^{x+f} \cdot d_x = v^{x+f} \cdot (l_x - l_{x+1})
$$

where $f$ is the mortality placement fraction controlled by
`config.mortality_placement`: `"beginning"` → $f = 0$; `"mid"` → $f = 0.5$ (**default**);
`"end"` → $f = 1$.
The classical actuarial formula $C_x = v^{x+1} d_x$ (end-of-year) corresponds to
`"end"`.  The commutation identity $A_x = M_x / D_x$ holds **exactly** for any
placement: $M_x = \sum C_k$ uses the same offset $f$ throughout, so different
placements yield internally consistent but numerically distinct insurance values
(representing payment timing at $k+f$, not approximations of each other).

### M function (sum of C)

$$
M_x = \sum_{k=x}^{\omega} C_k
$$

### R function (sum of M)

$$
R_x = \sum_{k=x}^{\omega} M_k
$$

(lx-function-person-years-lived)=
### Lx function (person-years lived)

At integer ages (uses `lt.Lx(x)`):

$$
L_x = \int_0^1 l_{x+t}\,\mathrm{d}t \approx \frac{l_x + l_{x+1}}{2} = l_x - \frac{d_x}{2}
$$

Exact under UDD. Requires integer age; fractional age raises `ValueError`.

At fractional ages (uses `lt.Lx_continuous(x+s, m=)`):

$$
L_{x+s} = \int_{x+s}^{x+s+1} l_t\,\mathrm{d}t
$$

Evaluated via the trapezoidal rule with $m+1$ equally-spaced nodes (default $m=12$).
Requires a strictly fractional age; integer age raises `ValueError`.

### Tx function (total future person-years)

At integer ages (uses `lt.Tx(x)`):

$$
T_x = \sum_{k=x}^{\omega-1} L_k
$$

Used to compute life expectancy: $\mathring{e}_x = T_x / l_x$. Requires integer age.

At fractional ages (uses `lt.Tx_continuous(x+s, m=)`):

$$
T_{x+s} = \int_{x+s}^{\omega} l_t\,\mathrm{d}t
$$

Evaluated via the trapezoidal rule (default $m=12$). Requires a strictly fractional age;
integer age raises `ValueError`. Used internally by `ex_continuous`.

(formulas-annuities)=
(life-annuities)=
## Life annuities

### Whole life annuity due

Present value of annual payments of 1 at the beginning of each year while the annuitant survives:

$$
\ddot{a}_x = \sum_{k=0}^{\omega - x} v^k \cdot {}_k p_x = \frac{N_x}{D_x}
$$

### Temporary annuity due

Present value for $n$ years:

$$
\ddot{a}_{x:\overline{n}|} = \sum_{k=0}^{n-1} v^k \cdot {}_k p_x = \frac{N_x - N_{x+n}}{D_x}
$$

### Deferred annuity

Annuity starting after $d$ years:

$$
{}_{d|}\ddot{a}_x = v^d \cdot {}_d p_x \cdot \ddot{a}_{x+d} = \frac{N_{x+d}}{D_x}
$$

### Deferred temporary annuity

Annuity of at most $n$ payments starting after a $d$-year deferral period:

$$
{}_{d|}\ddot{a}_{x:\overline{n}|}^{(m)}
  = \ddot{a}_{x:\overline{d+n}|}^{(m)} - \ddot{a}_{x:\overline{d}|}^{(m)}
$$

Equivalently, using the pure endowment:

$$
{}_{d|}\ddot{a}_{x:\overline{n}|}^{(m)} = {}_d E_x \cdot \ddot{a}_{x+d:\overline{n}|}^{(m)}
$$

In Lactuca, pass `d=` and `n=` together: `lt.äx(x, n=n, d=d, ir=ir)`.

### m-thly annuity due

Payments $m$ times per year:

$$
\ddot{a}_x^{(m)} = \frac{1}{m}\sum_{k=0}^{m(\omega-x)-1} v^{k/m} \cdot {}_{k/m} p_x
$$

Using Woolhouse approximation (first order):

$$
\ddot{a}_x^{(m)} \approx \ddot{a}_x - \frac{m-1}{2m}
$$

### Immediate annuity

Payments at the end of each period ($m = 1$, whole-life):

$$
a_x = \ddot{a}_x - 1
$$

For $m$-thly frequency the relationship generalises to:

$$
a_x^{(m)} = \ddot{a}_x^{(m)} - \frac{1}{m}
$$

$$
a_{x:\overline{n}|}^{(m)} = \ddot{a}_{x:\overline{n}|}^{(m)} - \frac{1}{m}\,(1 - {}_n E_x)
$$

These follow directly from Lactuca’s payment-grid definition: the first immediate payment
falls at $1/m$ rather than $0$, shifting every payment by $1/m$ relative to the
annuity-due.

### Continuous annuity

$$
\bar{a}_x = \int_0^{\omega-x} v^t \cdot {}_t p_x \, dt
$$

Approximation used by `continuous_simplified` mode
(see {ref}`continuous-simplified-formulas` for the temporary variant):

$$
\bar{a}_x \approx \ddot{a}_x - \frac{1}{2}
$$

(formulas-insurances)=
(life-insurances)=
## Life insurances

### Whole life insurance (discrete)

Present value of 1 payable on death, discounted at the mortality placement offset:

$$
A_x = \sum_{k=0}^{\omega-x-1} v^{k+f} \cdot {}_k p_x \cdot q_{x+k} = \frac{M_x}{D_x}
$$

where $f$ is the mortality placement fraction (see [$C_x$](#c-function-deaths)):
`"mid"` → $f = 0.5$ (**default**); `"end"` → $f = 1$; `"beginning"` → $f = 0$.
The classical textbook formula uses $v^{k+1}$ and corresponds to `"end"` placement.
The commutation identity $A_x = M_x/D_x$ holds for any placement because Lactuca's
$M_x = \sum C_k$ uses the same offset $f$ throughout.

### Term insurance

Insurance payable only if death occurs within $n$ years:

$$
A^1_{x:\overline{n}|} = \frac{M_x - M_{x+n}}{D_x}
$$

### Pure endowment

Payment only if the insured survives $n$ years:

$$
{}_n E_x = v^n \cdot {}_n p_x = \frac{D_{x+n}}{D_x}
$$

### Endowment insurance

Combination of term insurance and pure endowment:

$$
A_{x:\overline{n}|} = A^1_{x:\overline{n}|} + {}_n E_x
$$

### Continuous whole life insurance

$$
\bar{A}_x = \int_0^{\omega-x} v^t \cdot {}_t p_x \cdot \mu_{x+t} \, dt
$$

Classical UDD approximation (theoretical reference; not the formula Lactuca's
`continuous_simplified` mode uses — see {ref}`continuous-simplified-formulas`):

$$
\bar{A}_x \approx \frac{i}{\delta} A_x
$$

where $\delta = \ln(1+i)$ is the force of interest.

### Deferred insurance

Present value of a 1-unit benefit payable on death *after* a deferment period of $d$ years:

$$
{}_{d|}A_x = {}_d E_x \cdot A_{x+d}
$$

For a deferred term insurance (benefit paid only if death occurs within $n$ years after the
deferment period):

$$
{}_{d|}A^1_{x:\overline{n}|} = {}_d E_x \cdot A^1_{x+d:\overline{n}|}
$$

In Lactuca: `lt.Ax(x, d=d)` and `lt.Ax(x, d=d, n=n)`.  See {doc}`user_guide/deferment`
for code examples across all products.

(formulas-annuity-insurance)=
(relationship-between-annuities-and-insurances)=
## Relationship between annuities and insurances

:::{note}
**Notation clash — `d` has two meanings on this page.**
In formulas below, $d = i/(1+i)$ is the **discount rate** (a fixed finance constant).
Elsewhere on this page (deferred annuities, Lactuca’s `d=` argument) $d$ is the
**deferral period in years**.  The two quantities are always typographically identical
in standard actuarial notation; context determines which is meant.
:::

Fundamental relationship:

$$
\ddot{a}_x = \frac{1 - A_x}{d}
$$

where $d = \frac{i}{1+i}$ is the discount rate.

For temporary annuity:

$$
\ddot{a}_{x:\overline{n}|} = \frac{1 - A_{x:\overline{n}|}}{d}
$$

For continuous modes the analogous relationship uses the force of interest
$\delta = \ln(1+i)$:

$$
\bar{a}_x = \frac{1 - \bar{A}_x}{\delta}
$$

:::{note}
The discrete identities $\ddot{a}_x = (1-A_x)/d$ and $\ddot{a}_{x:\overline{n}|} = (1-A_{x:\overline{n}|})/d$
are classical algebraic results that hold when $A_x$ is the **end-of-year** insurance
(i.e., `config.mortality_placement = "end"`).  Under Lactuca's default `"mid"`
placement, $A_x$ represents a mid-year benefit and these exact equalities do not hold,
though the approximation error is small in practice.
:::

(formulas-life-expectancy)=
(life-expectancy)=
## Life expectancy

### Complete life expectancy

The **complete (entire) expectation of life** $\mathring{e}_x$ is the expected total
future lifetime of a life aged $x$:

$$
\mathring{e}_x = \int_0^{\omega-x} {}_t p_x \, dt
$$

:::{warning}
**`ex(x)` requires an integer age.**
Passing a fractional age raises `ValueError`.
:::

**`ex(x)` — integer ages only**: uses the UDD discrete approximation
$T_x = \sum L_k$:

$$
T_x = \sum_{k=x}^{\omega-1} L_k \approx \sum_{k=x}^{\omega-1} \frac{l_k + l_{k+1}}{2},
\qquad \mathring{e}_x = \frac{T_x}{l_x}
$$

:::{warning}
**`ex_continuous(x + s, m=)` requires a strictly fractional age.**
Passing an integer age raises `ValueError`.  Default `m = 12`.
:::

**`ex_continuous(x + s, m=)` — fractional ages only**: integrates $l_t$ numerically
(trapezoidal rule with $m$ sub-intervals per year):

$$
T_{x+s} = \int_{x+s}^{\omega} l_t \, dt, \qquad \mathring{e}_{x+s} = \frac{T_{x+s}}{l_{x+s}}
$$

### Curtate life expectancy

Expected whole years of future life:

$$
e_x = \sum_{k=1}^{\omega-x} {}_k p_x
$$

Under UDD the two expectations are related by:

$$
\mathring{e}_x = e_x + \tfrac{1}{2}
$$

(interest-rate-conversions)=
## Interest rate conversions

### Nominal to effective rate (m-thly)

$$
i = \left(1 + \frac{i^{(m)}}{m}\right)^m - 1
$$

### Effective to m-thly nominal rate

$$
i^{(m)} = m\left[(1+i)^{1/m} - 1\right]
$$

### Effective to force of interest

$$
\delta = \ln(1 + i)
$$

### m-thly discount rate

$$
d^{(m)} = m \left[1 - (1+i)^{-1/m}\right]
$$

(growth-rates-escalating-benefits)=
## Growth rates (escalating benefits)

For benefits increasing at rate $g$ per annum, the present value factor becomes:

$$
\text{PV factor} = \left(\frac{1+g}{1+i}\right)^t = v_g^t
$$

where $v_g = \frac{1+g}{1+i}$ is the adjusted discount factor.

Escalating whole-life annuity-due (annual payments, $m = 1$):

$$
\ddot{a}_x^{(g)} = \sum_{k=0}^{\omega-x} (1+g)^k \cdot v^k \cdot {}_k p_x
$$

For fractional payment frequency ($m > 1$), Lactuca uses an anniversary-based
growth index: the $j$-th payment receives factor $(1+g)^{\lfloor j/m \rfloor}$, so
that growth increments occur only at whole-year policy anniversaries regardless of
payment density. The complete $m$-thly growing annuity formulas are in
{doc}`user_guide/growth_conventions`.

(multiple-decrements)=
## Multiple decrements

:::{note}
**Lactuca assumes independent decrements** in all multiple-decrement calculations
(`DecrementTable`, `DisabilityTable`, `ExitTable`).  The independence formula
${}_{t}p_x^{(\tau)} = \prod_j {}_t p_x^{(j)}$ is applied throughout.  The general
inclusion-exclusion form below is shown for mathematical completeness only.
:::

For independent decrements $j = 1, 2, \ldots, n$:

$$
{}_t q_x^{(\tau)} = \sum_{j=1}^n {}_t q_x^{(j)} - \sum_{i<j} {}_t q_x^{(i,j)} + \cdots
$$

Under independence:

$$
{}_t p_x^{(\tau)} = \prod_{j=1}^n {}_t p_x^{(j)}
$$

Single decrement probability (cause $j$):

$$
{}_t q_x^{(j)} = \int_0^t {}_s p_x^{(\tau)} \cdot \mu_x^{(j)}(s) \, ds
$$

(joint-life-and-last-survivor)=
## Joint life and last survivor

### Joint life status

:::{note}
`LifeTable` assumes **statistical independence** of lives in all joint calculations
(`äxy`, `Axy`, `äxyz`, `Axyz`, etc.).
Correlated mortality is not supported.
:::

Survival of both lives:

$$
{}_t p_{xy} = {}_t p_x \cdot {}_t p_y \quad \text{(independence)}
$$

Joint life annuity:

$$
\ddot{a}_{xy} = \sum_{k=0}^{\infty} v^k \cdot {}_k p_{xy}
$$

### Last survivor status

At least one life survives:

$$
{}_t p_{\overline{xy}} = {}_t p_x + {}_t p_y - {}_t p_{xy}
$$

Last survivor annuity:

$$
\ddot{a}_{\overline{xy}} = \ddot{a}_x + \ddot{a}_y - \ddot{a}_{xy}
$$

### Joint life insurance and pure endowment

Joint life insurance — benefit on first death (uses `config.mortality_placement` offset $f$,
same convention as single-life `Ax`):

$$
A_{xy} = \sum_{k=0}^{\infty} v^{k+f} \cdot {}_k p_{xy} \cdot (1 - p_{x+k}\,p_{y+k})
$$

Joint pure endowment — both lives survive $n$ years:

$$
{}_n E_{xy} = v^n \cdot {}_n p_{xy} = v^n \cdot {}_n p_x \cdot {}_n p_y
$$

### Three-life joint status

Under independence of three lives:

$$
{}_t p_{xyz} = {}_t p_x \cdot {}_t p_y \cdot {}_t p_z
$$

Three-life joint annuity-due, insurance, and pure endowment:

$$
\ddot{a}_{xyz} = \sum_{k=0}^{\infty} v^k \cdot {}_k p_{xyz}
$$

$$
A_{xyz} = \sum_{k=0}^{\infty} v^{k+f} \cdot {}_k p_{xyz} \cdot
  (1 - p_{x+k}\,p_{y+k}\,p_{z+k})
$$

$$
{}_n E_{xyz} = v^n \cdot {}_n p_x \cdot {}_n p_y \cdot {}_n p_z
$$

(formulas-by-calculation-mode)=
## Formulas by calculation mode

Lactuca provides four calculation modes (set via `config.calculation_mode`). The
same method (`äx`, `Ax`, etc.) uses different formulae depending on the active mode.
Full details in {doc}`user_guide/calculation_modes`.

:::{note}
Change mode globally with `config.calculation_mode = "<mode>"`.  Allowed values:
`"discrete_precision"` (default), `"discrete_simplified"`,
`"continuous_precision"`, `"continuous_simplified"`.
Any other string raises `ValueError`.
Call `config.reset()` to restore the default.
:::

### `discrete_precision` — integer ages, $m = 1$

Closed-form equivalents (direct summation reduces to commutation formulas for
integer ages and $m=1$).  All identities hold **exactly** for any value of
`config.mortality_placement` (see [C function](#c-function-deaths) note above):

$$\ddot{a}_{x:\overline{n}|} = \frac{N_x - N_{x+n}}{D_x}, \qquad \ddot{a}_x = \frac{N_x}{D_x}$$

$$A^1_{x:\overline{n}|} = \frac{M_x - M_{x+n}}{D_x}, \qquad A_x = \frac{M_x}{D_x}$$

$${}_{n}E_x = \frac{D_{x+n}}{D_x}$$

### `discrete_precision` — $m > 1$ or fractional age

Exact summation over the payment grid, using $\ell_x$ interpolation at each grid
point.

Annuity-due:

$$\ddot{a}_{x:\overline{n}|}^{(m)} = \frac{1}{m} \sum_{j=0}^{mn-1} v^{j/m} \cdot {}_{j/m}p_x$$

Annuity-immediate:

$$a_{x:\overline{n}|}^{(m)} = \frac{1}{m} \sum_{j=0}^{mn-1} v^{(j+1)/m} \cdot {}_{(j+1)/m}p_x$$

Insurance ($m$-thly):

$$A_x^{(m)} = \sum_{k=0}^{mn-1} v^{k/m\,+\,f/m} \cdot {}_{k/m}p_x \cdot {}_{1/m}q_{x+k/m}$$

where $f/m$ is the mortality placement micro-offset within each $1/m$-year sub-period
($f \in \{0,\,0.5,\,1\}$ from `config.mortality_placement`; see [C function](#c-function-deaths)).

Survival and death probabilities are obtained by interpolating $\ell_x$ with the
method set in `config.lx_interpolation` (see {doc}`user_guide/lx_interpolation`).

### `discrete_simplified`

Classical Woolhouse first-order approximation converting $m=1$ to frequency $m$:

$$\ddot{a}_{x:\overline{n}|}^{(m)} \approx \ddot{a}_{x:\overline{n}|} - \frac{m-1}{2m}\,(1 - {}_{n}E_x)$$

$$\ddot{a}_x^{(m)} \approx \ddot{a}_x - \frac{m-1}{2m}$$

Woolhouse approximation for insurances:

$$A_x^{(m)} \approx A_x + \frac{m-1}{2m}\left(A_{x+1} - A_x\right)$$

### `continuous_precision`

Numerical integration over the unrounded $\ell_x$ curve (raw values):

$$\bar{a}_{x:\overline{n}|} = \int_0^n v^t \cdot {}_t p_x \, dt$$

$$\bar{A}^1_{x:\overline{n}|} = \int_0^n v^t \cdot {}_t p_x \cdot \mu_{x+t} \, dt$$

The force of mortality $\mu_{x+t}$ is derived numerically from the raw $\ell_x$ grid.
The approximation method is controlled by `config.force_mortality_method`; allowed
values: `"finite_difference"` (**default**), `"spline"`, `"kernel"`.
Passing any other string raises `ValueError`.

(continuous-simplified-formulas)=
### `continuous_simplified`

Arithmetic interpolation using the rounded $\ell_x$ grid:

$$\bar{a}_x \approx \ddot{a}_x - \frac{1}{2}$$

$$\bar{a}_{x:\overline{n}|} \approx \ddot{a}_{x:\overline{n}|} - \frac{1}{2}\,(1 - {}_n E_x)$$

$$\bar{A}_{x+s} \approx \frac{\bar{A}_x + \bar{A}_{x+1}}{2}, \quad 0 < s < 1$$

(select-and-ultimate-tables)=
## Select-and-ultimate tables

Some tables are **select** tables: the mortality rate $q_{[x]+k}$ depends not only
on the attained age $x + k$ but also on the time elapsed since selection $k$ and the
age at selection $x$.  Once the select period has elapsed, the rate reverts to the
**ultimate** table which depends on attained age only.
See [Select-ultimate notation](#formulas-select-notation) for the mathematical definition.

Lactuca reads the select period from the table's `select_period` metadata field.
The fallback to ultimate mortality is applied automatically once duration reaches
`start_duration + select_period` years — no code change is required in the calling layer.

**Example — DAV 2004R Select-Ultimate (`select_period = 5`):**

```python
from lactuca import LifeTable

# Load select-ultimate table at duration 1 (freshly underwritten)
dav = LifeTable("DAV2004R_SelUlt_1o", "m", duration=1)
print(dav.select_period)      # 5

# Select mortality at age [60], 2 years after selection: q_{[60]+2}
dav.duration = 2
q_sel = dav.qx(60)

# Ultimate mortality at attained age 65: q_65
dav.duration = "ult"
q_ult = dav.qx(65)
```

For a full treatment see {doc}`user_guide/tables_taxonomy`.

(references)=
## References

### Actuarial mathematics

- Bowers, N.L., et al. (1997). *Actuarial Mathematics* (2nd ed.). Society of Actuaries.
- Dickson, D.C.M., Hardy, M.R., Waters, H.R. (2019). *Actuarial Mathematics for Life Contingent Risks* (3rd ed.). Cambridge University Press.
- Promislow, S.D. (2014). *Fundamentals of Actuarial Mathematics* (3rd ed.). Wiley.
- Gerber, H.U. (1997). *Life Insurance Mathematics* (3rd ed.). Springer / Swiss Association of Actuaries.

### Numerical methods and financial mathematics

- Shiu, E.S.W. (1984). Woolhouse's formula and other summation techniques. *Transactions of the Society of Actuaries*, 36.
- Norberg, R. (1990). *Reserve formula for annuities and insurances with the focus on multiple state models*. Scandinavian Actuarial Journal.
- Press, W.H., et al. (2007). *Numerical Recipes: The Art of Scientific Computing* (3rd ed.). Cambridge University Press.

### Multiple decrements and multiple-state models

- Haberman, S., Pitacco, E. (1999). *Actuarial Models for Disability Insurance*. Chapman & Hall/CRC.
- Pitacco, E., Denuit, M., Haberman, S., Olivieri, A. (2009). *Modelling Longevity Dynamics for Pensions and Annuity Business*. Oxford University Press.

### Mortality table sources

The bundled tables are sourced from the official publications of each jurisdiction's
supervising authority or professional association.  See {doc}`user_guide/bundled_tables`
for the complete list and provenance of each table.

---

**Note:** All formulas assume integer ages unless fractional age interpolation (UDD/CFM) is explicitly specified.

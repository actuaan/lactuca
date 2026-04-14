# Life Insurances

A **life insurance** pays a lump-sum benefit at — or shortly after — the death of the
insured.  Lactuca provides `lt.Ax` for death-benefit calculations and `lt.nEx` for
pure endowments.  Both are also importable as standalone functions from `lactuca`.

---

## Types

| Product | Pays when | Method | Notes |
|---------|-----------|--------|-------|
| Whole-life insurance | On any death | `lt.Ax(x)` | — |
| Term insurance | On death within $n$ years | `lt.Ax(x, n=n)` | `n=` required |
| Pure endowment | If $(x)$ survives $n$ years | `lt.nEx(x, n=n)` | no `d`, `m`, or `gr` |
| Endowment insurance | First of: death or survival at $n$ | `lt.Ax(x, n=n) + lt.nEx(x, n=n)` | death + survival components |

:::{note}
`lt.Ax(x, n=n)` returns only the **term (death) insurance** $A^1_{x:\overline{n}|}$ —
the benefit payable on death within $n$ years.  To obtain the full **endowment
insurance** $A_{x:\overline{n}|}$ (benefit paid on death *or* survival at $n$, whichever
occurs first), add the pure endowment:

```python
A_endowment = lt.Ax(x, n=n) + lt.nEx(x, n=n)
```
:::

---

## The insurance formula

The APV of a **term life insurance** at benefit-payment frequency $m$ is:

$$
A^1_{x:\overline{n}|}{}^{(m)}
= \sum_{j=0}^{mn-1} v^{j/m + \delta_m} \cdot {}_{j/m}p_x \cdot {}_{1/m}q_{x+j/m}
$$

Each term is the present value of the benefit (1 unit) payable at time $j/m + \delta_m$
if $(x)$ survives to the start of sub-period $j$ and then dies within it.  The offset
$\delta_m$ is controlled by `config.mortality_placement` (see below).

For a **whole-life** insurance (`n=None`) the sum extends to the limiting age $\omega$.

---

## Core parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `x` | `float` | *(required)* | Current attained age |
| `n` | `float \| None` | `None` | Term in years; `None` = whole life |
| `m` | `int` | `1` | Benefit payment frequency per year (1, 2, 3, 4, 6, 12, 14, 24, 26, 52, 365) |
| `d` | `float` | `0.0` | Deferment — benefit payable only if death occurs after $x + d$ |
| `ts` | `float` | `0.0` | Elapsed time for reserve calculations |
| `ir` | `float \| InterestRate \| None` | table default | Interest rate |
| `gr` | `float \| GrowthRate \| None` | `None` | Benefit growth rate (escalating sum assured) |

`Ax` is also importable as a functional-style wrapper:

```python
from lactuca import Ax
```

`nEx` does **not** accept `d`, `m`, or `gr`.  It accepts `ts` for reserve calculations.

---

## Mortality placement and payment timing

`config.mortality_placement` determines the sub-period timing of the death benefit
($\delta_m$ in the formula above).  It affects **insurances only** — annuities and
pure endowments are unaffected.

| Setting | $\delta_m$ | Convention |
|---------|-----------|------------|
| `"mid"` (default) | $\tfrac{1}{2m}$ | Death at mid-period (UDD mid-year) |
| `"beginning"` | $0$ | Payment immediately on death |
| `"end"` | $\tfrac{1}{m}$ | Traditional end-of-period convention |

```python
from lactuca import config

config.mortality_placement = "mid"       # default
config.mortality_placement = "end"       # traditional commutation convention
config.mortality_placement = "beginning" # payment immediately on death
config.reset()                           # restore default ("mid")
```

For the full reference and interaction with calculation modes, see
{ref}`mortality placement <mortality-placement>` in {doc}`calculation_modes`.

---

## Equivalence with life annuities

The whole-life insurance and the annuity-due satisfy the **annuity-insurance duality**,
which follows from the total probability law.  At annual frequency ($m = 1$) with
end-of-year convention:

$$
A_x + d\,\ddot{a}_x = 1
\qquad \Longleftrightarrow \qquad
A_x = 1 - d\,\ddot{a}_x
$$

where $d = 1 - v = i/(1+i)$ is the annual effective discount rate.

For a finite-term endowment:

$$
A_{x:\overline{n}|} + d\,\ddot{a}_{x:\overline{n}|} = 1
\qquad \Longleftrightarrow \qquad
A_{x:\overline{n}|} = 1 - d\,\ddot{a}_{x:\overline{n}|}
$$

:::{note}
The identity is exact for $m = 1$ and `mortality_placement = "end"` (traditional
commutation-function convention).  With the default `"mid"` placement, use the
$m$-thly discount rate $d^{(m)} = m(1 - v^{1/m})$ for the corresponding $m$-thly
version.  Small numerical discrepancies generally arise because the two quantities
are computed by independent engine dispatches.
:::

---

## Commutation function connection

For integer ages, $m = 1$, and `mortality_placement = "end"`:

$$
A_x = \frac{M_x}{D_x}
$$

See {doc}`commutation_functions` for the algebraic derivation.  The calculation
engine used by `Ax` evaluates the defining sum directly — it does **not** use
commutation functions — so it correctly handles all frequencies, fractional ages,
and piecewise interest rate curves.

---

## Pure endowment

The pure endowment ${}_{n}E_x$ is the APV of 1 unit payable at time $n$ if $(x)$ survives:

$$
{}_{n}E_x = v^n \cdot {}_{n}p_x
$$

It is a key building block: the endowment insurance is the term insurance plus the
pure endowment, and the factorisation of deferred annuities uses it explicitly (see
{doc}`deferment`).  It does **not** accept `d`, `m`, or `gr`.

---

## Code examples

### Whole-life, term, and endowment insurance

```python
from lactuca import LifeTable, config

config.decimals.insurances = 6
lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)

# Whole-life insurance:  Ax(65)
Ax_wl = lt.Ax(65)
print(Ax_wl)

# 20-year term insurance:  A¹_{65:20|}
Ax_term = lt.Ax(65, n=20)
print(Ax_term)                   # < Ax_wl (deaths after 20 years excluded)

# 20-year pure endowment:  20E65
nEx_20 = lt.nEx(65, n=20)
print(nEx_20)

# 20-year endowment insurance:  A_{65:20|} = A¹ + 20E
endowment = round(Ax_term + nEx_20, 6)
print(endowment)                 # payment is certain; APV < 1 due to discounting

config.reset()
```

### Monthly benefit insurance (`m=12`)

```python
from lactuca import LifeTable, config

config.decimals.insurances = 6
lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)

# Annual insurance (default m=1)
Ax_annual = lt.Ax(65)

# Monthly benefit: benefit paid at mid-month of death (default "mid" placement)
Ax_monthly = lt.Ax(65, m=12)

# Monthly benefit is slightly higher: benefit paid sooner (less discounting)
print(Ax_annual < Ax_monthly)    # True

config.reset()
```

### Annuity-insurance duality

```python
from lactuca import LifeTable, config

config.decimals.annuities = 6
config.decimals.insurances = 6
lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)

i = 0.03
d_rate = i / (1 + i)    # annual discount rate d = 1 - v

Ax_val = lt.Ax(65)      # whole-life insurance (m=1, default "mid" placement)
ax_val = lt.äx(65)      # whole-life annuity-due (m=1)

# Check: Ax + d * äx ≈ 1  (near-exact under m=1, with small gap from "mid" placement)
print(round(Ax_val + d_rate * ax_val, 4))   # ≈ 1.0

config.reset()
```

### Deferred insurance

A deferred insurance pays only if death occurs after the deferment period.  See
{doc}`deferment` for the full treatment:

```python
from lactuca import LifeTable, config

config.decimals.insurances = 6
lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)

# Standard whole-life insurance
Ax = lt.Ax(65)

# 5-year deferred: only deaths after age 70 trigger the benefit
Ax_def = lt.Ax(65, d=5)

print(Ax > Ax_def)     # True — deaths in [0, 5) excluded

config.reset()
```

### Prospective reserve at elapsed time `ts`

```python
from lactuca import LifeTable, config

config.decimals.insurances = 6
lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)

x, n = 55, 30

# APV at issue (ts=0)
Ax_issue = lt.Ax(x, n=n)

# APV after 10 years: effective age 65, remaining term 20
Ax_ts = lt.Ax(x, n=n, ts=10)
Ax_direct = lt.Ax(65, n=20)

# Both approaches give the same value
print(round(Ax_ts - Ax_direct, 8))   # → 0.0

config.reset()
```

---

## See also

- {doc}`notation_glossary` — actuarial symbols and notation
- {doc}`life_annuities_guide` — `äx`, `ax` and the annuity-insurance duality
- {doc}`commutation_functions` — $M_x/D_x$ formula and commutation function reference
- {doc}`deferment` — deferment parameter `d`: deferred insurances with factorisation proof
- {doc}`prospective_reserve` — net level premium $P = A_x / \ddot{a}_x$, prospective reserve formula, and the `ts` parameter
- {doc}`calculation_modes` — mortality placement, discrete vs. continuous modes
- {doc}`joint_life_calculations` — joint-life insurances `Axy`, first-death insurance
- {doc}`interest_rates_guide` — `InterestRate` class and term structures

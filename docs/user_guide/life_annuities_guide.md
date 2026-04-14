# Life Annuities

A **life annuity** is a series of payments made as long as a designated life survives.
It is the primary building block in individual and occupational pensions, annuity pricing,
premium calculations, and prospective reserve evaluation.

In Lactuca, single-life life annuities are computed via `lt.äx` (annuity-due, prepayable)
and `lt.ax` (annuity-immediate, postpayable).  Both are also importable as standalone
functions directly from `lactuca`.

---

## Annuity-due and annuity-immediate

**Annuity-due** (`äx`): a payment of $1/m$ is made **at the start** of each sub-annual
period, conditional on the life being alive.  The first payment is at $t = 0$ (or after
the deferment $d$ if specified).

**Annuity-immediate** (`ax`): a payment of $1/m$ is made **at the end** of each
sub-annual period, conditional on the life surviving to that point.  The first payment
falls at $t = 1/m$.

The two forms are related by:

$$
\ddot{a}_{x:\overline{n}|}^{(m)}
= a_{x:\overline{n}|}^{(m)} + \frac{1}{m}\!\left(1 - {}_{n}E_x\right)
$$

For whole life ($n \to \infty$, ${}_{n}E_x \to 0$) this simplifies to
$\ddot{a}_x^{(m)} = a_x^{(m)} + \tfrac{1}{m}$.

---

## The annuity formula

The actuarial present value (APV) of a **temporary life annuity-due** at frequency $m$ is:

$$
\ddot{a}_{x:\overline{n}|}^{(m)}
= \frac{1}{m} \sum_{j=0}^{mn-1} v^{j/m} \cdot {}_{j/m}p_x
$$

Each term discounts a payment of $1/m$ payable at time $j/m$ (start of the $j$-th
sub-period) by the discount factor $v^{j/m} = (1+i)^{-j/m}$ and weights it by the
survival probability ${}_{j/m}p_x$ that $(x)$ reaches time $j/m$.

For a **whole-life** annuity (`n=None`), the sum runs to the limiting age $\omega$:

$$
\ddot{a}_x^{(m)} = \frac{1}{m} \sum_{j=0}^{m(\omega-x)-1} v^{j/m} \cdot {}_{j/m}p_x
$$

The corresponding **annuity-immediate** formula:

$$
a_{x:\overline{n}|}^{(m)}
= \frac{1}{m} \sum_{j=1}^{mn} v^{j/m} \cdot {}_{j/m}p_x
$$

---

## Core parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `x` | `float` | *(required)* | Current attained age |
| `n` | `float \| None` | `None` | Term in years; `None` = whole life |
| `m` | `int` | `1` | Payment frequency per year (1, 2, 3, 4, 6, 12, 14, 24, 26, 52, 365) |
| `d` | `float` | `0.0` | Deferment period in years |
| `ts` | `float` | `0.0` | Elapsed time for reserve calculations |
| `ir` | `float \| InterestRate \| None` | table default | Interest rate |
| `gr` | `float \| GrowthRate \| None` | `None` | Benefit growth rate (escalation) |

`äx` and `ax` are importable as functional-style wrappers:

```python
from lactuca import äx, ax
```

See {doc}`deferment` for the full treatment of `d`, and {doc}`prospective_reserve`
for `ts`.

---

## Equivalence with life insurances

The annuity-due and whole-life insurance are linked through the **annuity-insurance
duality**.  At annual frequency ($m = 1$) with end-of-year death benefit:

$$
A_x + d\,\ddot{a}_x = 1
\qquad \Longleftrightarrow \qquad
\ddot{a}_x = \frac{1 - A_x}{d}
$$

where $d = 1 - v = i/(1+i)$ is the annual effective discount rate.

For a temporary annuity and matching endowment insurance of the same term $n$:

$$
A_{x:\overline{n}|} + d\,\ddot{a}_{x:\overline{n}|} = 1
\qquad \Longleftrightarrow \qquad
\ddot{a}_{x:\overline{n}|} = \frac{1 - A_{x:\overline{n}|}}{d}
$$

:::{note}
In Lactuca, `lt.Ax(x, n=n)` returns only the **term (death) insurance**
$A^1_{x:\overline{n}|}$.  The full endowment insurance is
`lt.Ax(x, n=n) + lt.nEx(x, n=n)`.

The duality holds exactly for $m = 1$ with `config.mortality_placement = "end"`.
For other modes (especially `"mid"`, the default) and $m > 1$, use the
$m$-thly discount rate $d^{(m)} = m(1 - v^{1/m})$ in place of $d$.  Small
numerical discrepancies will appear because the two quantities are computed by
independent engine dispatches.
:::

---

## Commutation function connection

For integer ages and annual frequency ($m = 1$) the standard commutation identity is:

$$
\ddot{a}_x = \frac{N_x}{D_x}
$$

See {doc}`commutation_functions` for the algebraic derivation.  The calculation engine
used by `äx` and `ax` evaluates the defining sum directly — it does **not** use
commutation functions — so it correctly handles all frequencies, fractional ages,
and piecewise interest rate curves.

---

## Actuarial contexts

| Actuarial use case | Method call | Key parameters |
|--------------------|-------------|----------------|
| Annual retirement pension (life) | `lt.äx(x)` | `m=12` for monthly payments |
| Pension deferred until age 65 | `lt.äx(x, d=65-x)` | `d = 65 − current age` |
| Premium-paying temporary annuity | `lt.äx(x, n=n_premium)` | — |
| Widow's reversionary pension | `lt.äx(y) - lt.äxy([x, y], ...)` | see {doc}`joint_life_calculations` |
| Escalating pension (CPI-linked) | `lt.äx(x, gr=GrowthRate(0.02))` | see {doc}`growth_rates_guide` |
| Prospective reserve at time $t$ | `lt.äx(x, n=n, ts=t)` | see {doc}`prospective_reserve` |

---

## Code examples

### Whole-life and temporary annuities

```python
from lactuca import LifeTable, config

config.decimals.annuities = 4
lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)

# Annual whole-life annuity-due: äx(65)
a_wl = lt.äx(65)
print(a_wl)           # → 16.0899

# 20-year temporary annuity-due: ä_{65:20|}
a_temp = lt.äx(65, n=20)
print(a_temp)         # < a_wl (truncated at n=20)

# Monthly temporary annuity-due: ä^(12)_{65:20|}
a_m12 = lt.äx(65, n=20, m=12)
print(a_m12)          # lower than a_temp: each sub-annual payment is weighted by exact fractional-age survival probability

# Annuity-immediate (postpayable) equivalent: a_{65:20|}
a_imm = lt.ax(65, n=20)
print(a_imm)          # < a_temp by approximately 1/m × (1 - nEx)

config.reset()
```

### Due-to-immediate conversion

```python
from lactuca import LifeTable, config

config.decimals.annuities = 4
lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)

n = 20
x = 65
a_due = lt.äx(x, n=n)
nEx   = lt.nEx(x, n=n)
a_imm = lt.ax(x, n=n)

# Identity: ä = a + (1/m) * (1 - nEx)  for m=1
print(round(a_due - a_imm, 6))           # ≈ 1 - nEx  (for m=1; smaller when nEx is large)
print(round(a_due - (a_imm + 1 - nEx), 6))  # ≈ 0.0

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

ax_val = lt.äx(65)      # annuity-due, m=1
Ax_val = lt.Ax(65)      # annual whole-life insurance

# Check: Ax + d * äx ≈ 1  (exact under m=1, mortality_placement="end")
print(round(Ax_val + d_rate * ax_val, 4))   # ≈ 1.0 (small gap from default "mid" placement)

config.reset()
```

### Prospective reserve at elapsed time `ts`

```python
from lactuca import LifeTable, config

config.decimals.annuities = 4
lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)

x, n = 55, 30

# APV at issue (ts=0)
a_issue = lt.äx(x, n=n)
print(a_issue)

# Prospective reserve after 10 years:
# effective age = 65, remaining term = 20  — identical approaches:
a_ts = lt.äx(x, n=n, ts=10)        # using ts parameter
a_direct = lt.äx(65, n=20)         # calling directly at attained age

print(round(a_ts - a_direct, 8))   # → 0.0

config.reset()
```

---

## See also

- {doc}`notation_glossary` — actuarial symbols and notation
- {doc}`life_insurances_guide` — `Ax`, `nEx` and the annuity-insurance duality
- {doc}`commutation_functions` — $N_x/D_x$ formula and commutation function reference
- {doc}`deferment` — deferment parameter `d`: deferred annuities with proof of factorisation
- {doc}`prospective_reserve` — net level premium $P = A_x / \ddot{a}_x$, prospective reserve, and the `ts` parameter for off-anniversary reserves
- {doc}`calculation_modes` — discrete vs. continuous and payment frequency effects
- {doc}`joint_life_calculations` — joint-life annuities `äxy`, reversionary annuity
- {doc}`growth_rates_guide` — growing annuities with `GrowthRate`
- {doc}`interest_rates_guide` — `InterestRate` class and term structures
- {doc}`lx_interpolation` — fractional-age survival under UDD and constant force

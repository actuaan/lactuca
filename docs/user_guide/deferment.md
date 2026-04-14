# Deferred Life Contingencies

A *deferred* life contingency is one in which the payment stream — whether an
annuity or an insurance benefit — does not start immediately but only after a waiting
period of $d$ years measured from the reference age $x$.  The parameter `d` controls
this waiting period uniformly across all life annuity and life insurance methods in
Lactuca.

:::{note}
This page covers `d` in isolation (the default case where the valuation date coincides
with the reference age, i.e. `ts=0`).  When combining deferment with an off-anniversary
time shift `ts`, the interaction rule $d_{\text{eff}} = \max(d - ts,\; 0)$ applies; see
{doc}`prospective_reserve` for a full treatment.
:::

---

## The deferment parameter

`d` is the number of years that must elapse — from age $x$ — before the first payment
is made.  Internally, every payment time is shifted by $d$:

$$
t_j = \frac{j + p_{\text{timing}}}{m} + d
$$

where $j$ counts payments from zero, $m$ is the annual payment frequency, and
$p_{\text{timing}}$ equals 0 for annuity-due (payments at start of each period) and
1 for annuity-immediate (payments at end of each period).

Key properties:

- **Default**: `d=0` — payments start immediately.
- **Type**: non-negative float; sub-year values such as `d=0.5` are fully supported.
  A negative value raises a `ValueError`.
- **Scope**: `d` is accepted by all life annuity methods (`äx`, `ax`, `äxy`, `axy`,
  `äxyz`, `axyz`), all life insurance methods (`Ax`, `Axy`, `Axyz`), and the financial
  annuity methods on `InterestRate` (`ä`, `a`).  Pure endowment methods (`nEx`, `nExy`,
  `nExyz`) do **not** accept `d`; see [Interaction with pure endowments](#interaction-with-pure-endowments).

---

## Standard formulas

### Deferred whole-life annuities

**Annuity-due** (prepayable, `äx`):

$$
{}_{d|}\ddot{a}_x^{(m)} = \frac{1}{m} \sum_{j=0}^{\infty} v^{j/m\,+\,d} \cdot {}_{j/m\,+\,d}p_x
  = \nEx{d}{x} \cdot \ddot{a}_{x+d}^{(m)}
$$

The factorisation uses the chain rule ${}_{j/m+d}p_x = \px{d}{x} \cdot {}_{j/m}p_{x+d}$
and separates $v^{j/m+d} = v^d \cdot v^{j/m}$:

$$
\frac{1}{m}\sum_{j=0}^{\infty} v^{j/m+d}\cdot {}_{j/m+d}p_x
= v^d\,\px{d}{x}\cdot\frac{1}{m}\sum_{j=0}^{\infty} v^{j/m}\cdot {}_{j/m}p_{x+d}
= \nEx{d}{x}\cdot\ddot{a}_{x+d}^{(m)}
$$

**Annuity-immediate** (postpayable, `ax`):

$$
{}_{d|}a_x^{(m)} = \frac{1}{m} \sum_{j=1}^{\infty} v^{j/m\,+\,d} \cdot {}_{j/m\,+\,d}p_x
  = \nEx{d}{x} \cdot a_{x+d}^{(m)}
$$

The factorisation via the pure endowment $\nEx{d}{x} = v^d \cdot \px{d}{x}$ is exact
for both forms: the pure endowment discounts survival to age $x + d$, and from that age
a standard whole-life annuity applies.  The two forms are related by

$$
{}_{d|}a_x^{(m)} = {}_{d|}\ddot{a}_x^{(m)} - \frac{1}{m}\,\nEx{d}{x}
$$

which is the classical due-to-immediate conversion shifted to the deferred origin.

### Deferred temporary annuities

**Annuity-due** (prepayable, `äx` with `n=`):

$$
{}_{d|}\ddot{a}_{x:\overline{n}|}^{(m)}
  = \ddot{a}_{x:\overline{d+n}|}^{(m)} - \ddot{a}_{x:\overline{d}|}^{(m)}
  = \nEx{d}{x} \cdot \ddot{a}_{x+d:\overline{n}|}^{(m)}
$$

**Annuity-immediate** (postpayable, `ax` with `n=`):

$$
{}_{d|}a_{x:\overline{n}|}^{(m)}
  = a_{x:\overline{d+n}|}^{(m)} - a_{x:\overline{d}|}^{(m)}
  = \nEx{d}{x} \cdot a_{x+d:\overline{n}|}^{(m)}
$$

In both cases pass `d=` and `n=` together: `lt.äx(x, n=n, d=d)` / `lt.ax(x, n=n, d=d)`.

The due-to-immediate relation for the deferred temporary case generalises to:

$$
{}_{d|}a_{x:\overline{n}|}^{(m)}
  = {}_{d|}\ddot{a}_{x:\overline{n}|}^{(m)}
    - \frac{1}{m}\left(\nEx{d}{x} - \nEx{d+n}{x}\right)
$$

The first pure-endowment term accounts for the payment at the start of the deferment
period (present in the due form, absent in the immediate) and the second for the
extra payment at the end of term $d+n$ (present in the immediate, absent in the due).

### Deferred life insurance

A deferred life insurance pays the benefit only if death occurs **after** the deferment
period ends:

$$
{}_{d|}A_x = \nEx{d}{x} \cdot A_{x+d}
$$

The same identity holds for a finite term:

$$
{}_{d|}A^1_{x:\overline{n}|} = \nEx{d}{x} \cdot A^1_{x+d:\overline{n}|}
$$

Pass `d=` (and optionally `n=`) to `lt.Ax(x, d=d)`.

---

## API examples

### Single-life annuities

```python
from lactuca import LifeTable, config

config.decimals.annuities = 4
lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)

# 10-year deferred whole-life annuity-due:  10|äx(55)
a_def_whole_due = lt.äx(55, d=10)
print(a_def_whole_due)    # → 11.3534

# 10-year deferred whole-life annuity-immediate:  10|ax(55)
a_def_whole_imm = lt.ax(55, d=10)
print(a_def_whole_imm)    # → 10.6478

# 10-year deferred, 20-year temporary annuity-due:  10|ä55:20|
a_def_temp_due = lt.äx(55, n=20, d=10)
print(a_def_temp_due)     # → 9.5844

# 10-year deferred, 20-year temporary annuity-immediate:  10|a55:20|
a_def_temp_imm = lt.ax(55, n=20, d=10)
print(a_def_temp_imm)     # → 9.1223

# For comparison: standard whole-life annuity at age 65 (no deferment)
a_reference = lt.äx(65)
print(a_reference)        # → 16.0899

config.reset()
```

The pure endowment factorisation produces the same result:

```python
from lactuca import LifeTable, config

config.decimals.annuities = 4
lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)

# Direct: 10|äx(55)
a_direct = lt.äx(55, d=10)
print(a_direct)                                     # → 11.3534

# Equivalent via pure endowment:  10Ex(55) × äx(65)
via_endowment = round(lt.nEx(55, n=10) * lt.äx(65), 4)
print(via_endowment)                                # → 11.3534

config.reset()
```

### Single-life insurance

`Ax` accepts `d` and all four calculation modes honour it.  The benefit is payable
on death only if death occurs after the deferment period:

```python
from lactuca import LifeTable, config

config.decimals.insurances = 6
lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)

# Standard whole-life insurance:  Ax(55)
print(lt.Ax(55))              # → 0.424462

# 10-year deferred whole-life insurance:  10|Ax(55)
print(lt.Ax(55, d=10))        # → 0.380524

# 5-year deferred, 20-year term insurance:  5|A¹55:20|
print(lt.Ax(55, n=20, d=5))   # → 0.144705

config.reset()
```

:::{note}
A deferred insurance always produces a **lower** present value than the non-deferred
equivalent: deaths during the deferment period $[0, d)$ are excluded from the benefit,
reducing the expected present value.
:::

### Fractional deferment

`d` is a float and any non-negative value is valid, including sub-year periods:

```python
from lactuca import LifeTable, config

config.decimals.annuities = 4
lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)

# 6-month deferment, 20-year temporary:  0.5|ä60:20|
a_half = lt.äx(60, d=0.5, n=20)
print(a_half)    # → 13.8892

config.reset()
```

Survival at the fractional age $x + d$ is evaluated under the configured interpolation
hypothesis (UDD by default — see {doc}`lx_interpolation`).

### Multiple payment frequencies with deferment

`d` and `m` are fully orthogonal: the payment grid is
$t_j = j/(m) + d$ for annuity-due, regardless of frequency:

```python
from lactuca import LifeTable, config

config.decimals.annuities = 4
lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)

# Monthly payments, 10-year deferred whole-life:  10|ä⁽¹²⁾x(55)
a_monthly = lt.äx(55, d=10, m=12)
print(a_monthly)    # → 11.0274

config.reset()
```

---

## Joint-life deferred contingencies

All joint-life annuity and insurance methods accept `d`:

```python
from lactuca import LifeTable, config

config.decimals.annuities = 4
config.decimals.insurances = 6
lt_m, lt_f = LifeTable("PASEM2020_Rel_1o", ["m", "f"], interest_rate=0.03)

# Standard joint-life annuity-due (no deferment):  äxy(60, 58)
a_joint = lt_m.äxy([60, 58], table_y=lt_f)
print(a_joint)           # → 16.7085

# 10-year deferred joint-life annuity-due:  10|äxy(60, 58)
a_joint_def = lt_m.äxy([60, 58], table_y=lt_f, d=10)
print(a_joint_def)       # → 8.2606

# 10-year deferred, 15-year temporary:  10|äxy:15|(60, 58)
a_joint_def_tmp = lt_m.äxy([60, 58], table_y=lt_f, d=10, n=15)
print(a_joint_def_tmp)   # → 6.9214

# 5-year deferred joint first-death insurance:  5|Axy(60, 58)
Axy_def = lt_m.Axy([60, 58], table_y=lt_f, d=5)
print(Axy_def)           # → 0.481055

config.reset()
```

For the independence assumption, last-survivor and reversionary annuity formulas, see
{doc}`joint_life_calculations`.

---

## Financial annuities without mortality

`InterestRate.ä` and `InterestRate.a` accept `d` with exactly the same semantics as
their life-table counterparts — no mortality is involved, so survival probabilities
are identically 1 and the present value is a pure discount calculation.  This is
useful for pricing fixed-income structures or for isolating the interest component
of a life annuity:

```python
from lactuca import InterestRate, config

config.decimals.annuities = 4
ir = InterestRate(0.03)

# 10-year annuity-due, starting immediately:
print(ir.ä(n=10))          # → 8.7861

# The same annuity deferred 5 years:
print(ir.ä(n=10, d=5))     # → 7.5789

# Annuity-immediate, 5-year deferred:
print(ir.a(n=10, d=5))     # → 7.3582

config.reset()
```

For a flat interest rate the scaling is exact:
$\ddot{a}(d, n) = v^d \cdot \ddot{a}(n)$, i.e.\ a 5-year deferment at 3% multiplies
the undeferrred value by $v^5 = 1.03^{-5} \approx 0.8626$, giving
approximately $0.8626 \times 8.7861 \approx 7.5789$.

---

## Deferment and benefit growth

When a `GrowthRate` is active, **`d` shifts payment times but does not advance the
growth anniversary index**.  The growth exponent always counts completed years from
the first payment of the stream — the deferment period $[0, d)$ is not counted.

For an annual annuity-due with constant geometric growth $g$:

| Payment at $t$ | Growth factor |
|---:|---:|
| $d$ | $(1+g)^0 = 1$ |
| $d+1$ | $(1+g)^1$ |
| $d+2$ | $(1+g)^2$ |
| $\vdots$ | $\vdots$ |

Because the growth schedule always restarts at the first payment of any call, the
pure-endowment factorisation extends unchanged to growing annuities:

$$
{}_{d|}\ddot{a}_x(g) = \nEx{d}{x} \cdot \ddot{a}_{x+d}(g)
$$

```python
from lactuca import LifeTable, GrowthRate, config

config.decimals.annuities = 4
lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)
gr = GrowthRate(0.02)

# Deferred growing annuity: growth index starts from first payment at age 65
a_deferred = lt.äx(55, d=10, gr=gr)
print(a_deferred)           # → 14.1698

# Equivalent factorisation: nEx(55, 10) × äx(65, gr=2%)
nex = lt.nEx(55, n=10)
a_factor = round(nex * lt.äx(65, gr=gr), 4)
print(a_factor)             # → 14.1698

config.reset()
```

:::{note}
This is **different** from `ts` (time shift): `ts=5` consumes the first 5 anniverary
years from the growth schedule, while `d=5` merely delays the first payment without
altering the growth starting point.  See {doc}`growth_conventions` for the complete
interaction between `gr`, `d`, and `ts`.
:::

---

(interaction-with-pure-endowments)=
## Interaction with pure endowments

Pure endowment methods (`nEx`, `nExy`, `nExyz`) do **not** accept a `d` parameter.
This is intentional: the pure endowment $\nEx{n}{x} = v^n \cdot \px{n}{x}$ already
has a fixed observation horizon $n$; adding deferment on top of it would simply be
equivalent to changing $n$.  To value a deferred pure endowment, increase `n` directly:

$$
{}_d E_x = \texttt{lt.nEx}(x,\; n=d)
\qquad
{}_{d+n} E_x = \texttt{lt.nEx}(x,\; n=d+n)
$$

The annuity factorisation ${}_{d|}\ddot{a}_x = \nEx{d}{x} \cdot \ddot{a}_{x+d}$ makes
this connection explicit: `nEx` and `äx` together reproduce the deferred annuity.

---

## Calculation modes

`d` shifts the payment grid independently of the calculation mode.  The table below
summarises how each mode integrates the shift:

| Mode | Discrete / continuous | Effect of `d` |
|---|---|---|
| `"discrete_precision"` (default) | Discrete exact summation | Payment grid $t_j = j/m + d$; each $t_j$ enters survival and discount directly |
| `"discrete_simplified"` | Woolhouse UDD approximation | Same grid shift; Woolhouse 2-term adjustment applied after |
| `"continuous_precision"` | Numerical integration | Integration starts at $t = d$; the grid spans $[d,\, d+n]$ |
| `"continuous_simplified"` | Trapezoidal approximation | Sub-calculations shifted by $d$; results averaged to approximate continuous integral |

Both continuous modes use unrounded $l_x$ values (full float64 precision, no intermediate rounding).

No configuration changes are needed when using `d` with any mode.  Switch modes via
`config.calculation_mode`; see {doc}`calculation_modes` for full details.

---

## See also

- {doc}`prospective_reserve` — interaction between `d` and the elapsed-time
  parameter `ts`; the effective deferment rule $d_{\text{eff}} = \max(d - ts,\; 0)$
- {doc}`growth_conventions` — how `d` interacts with `GrowthRate`: deferment shifts
  payment *times* but does not advance the benefit anniversary index
- {doc}`joint_life_calculations` — joint-life methods, independence assumption,
  last-survivor and reversionary formulas
- {doc}`../formulas` — full mathematical reference for deferred annuity formulas
- {doc}`calculation_modes` — discrete vs. continuous modes and their frequency effects
- {doc}`lx_interpolation` — interpolation hypotheses (UDD / CFM) used at fractional ages

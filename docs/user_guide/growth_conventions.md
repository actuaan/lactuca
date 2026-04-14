# Growth Rate Conventions

This page documents the anniversary-based growth convention used throughout Lactuca
for all actuarial functions that accept a `gr` (growth rate) parameter:

- Single-life: `äx`, `ax`, `Ax`
- Named joint-life shortcuts (2 and 3 lives): `äxy`, `axy`, `äxyz`, `axyz`, `Axy`, `Axyz`
- Generic N-life variants: `äjoint`, `ajoint`, `Afirst`

Pure endowments (`nEx`, `nExy`, `nExyz`) do **not** accept `gr=`.

## Anniversary index definition

For a payment stream with frequency $m$ (payments per year) starting at time $d$
(deferment), the $j$-th payment ($j = 0, 1, 2, \ldots$) receives the growth factor:

$$F(j) = (1 + g)^{\lfloor j / m \rfloor}$$

where $\lfloor j / m \rfloor$ is the **annual anniversary index** — the number of
complete years of payments that have elapsed before payment $j$. This formula is
for the default **geometric** growth type (`growth_type='g'`). For **arithmetic**
growth (`growth_type='a'`), the factor is $F(j) = 1 + g \cdot \lfloor j / m \rfloor$;
the anniversary index $\lfloor j / m \rfloor$ is the same in both types.

:::{note}
This formula assumes the default `apply_from_first=False`: the first payment has
factor 1 and growth starts from the second anniversary onward. With
`apply_from_first=True` the exponent becomes $\lfloor j/m \rfloor + 1$, so the
first payment already carries the full first-year growth. See {doc}`growth_rates_guide`
for the `apply_from_first` parameter.
:::

The absolute payment time for an **annuity-immediate** (`ax`) is:

$$t_j = \frac{j + 1}{m} + d$$

**The deferment $d$ shifts payment times but does not affect the anniversary
index.** The growth exponent always counts full-year anniversaries from the first
payment of the stream, regardless of $d$.

### Full formula (growing temporary life annuity-due)

$$\ddot{a}_{x:\overline{n}|}^{(m)}(g) = \frac{1}{m}\sum_{j=0}^{mn-1} (1+g)^{\lfloor j/m \rfloor}
    \cdot v^{j/m} \cdot {}_{j/m}p_x$$

### Full formula (growing temporary life annuity-immediate)

$$a_{x:\overline{n}|}^{(m)}(g) = \frac{1}{m}\sum_{j=1}^{mn} (1+g)^{\lfloor (j-1)/m \rfloor}
    \cdot v^{j/m} \cdot {}_{j/m}p_x$$

For the deferred version with deferment $d$:

$${}_{d|}a_{x:\overline{n}|}^{(m)}(g) = \frac{1}{m}\sum_{j=1}^{mn} (1+g)^{\lfloor (j-1)/m \rfloor}
    \cdot v^{j/m + d} \cdot {}_{j/m + d}p_x$$

Both `äx` and `ax` apply the same growth rule: the $k$-th payment (counting from 1)
carries factor $(1+g)^{\lfloor (k-1)/m \rfloor}$, so growth steps once per complete
policy year regardless of payment frequency. The index conventions differ only to
keep each sum naturally aligned with its payment times ($j = 0$ to $mn-1$ for
annuity-due, $j = 1$ to $mn$ for annuity-immediate).

### Continuous calculation modes

In **`continuous_precision`** mode the payment index $j$ is replaced by continuous
time. The growth factor applied at time $t$ with deferment $d$ is
$F(\lfloor t - d \rfloor)$: growth steps at each full-year anniversary of the
payment stream, regardless of payment density.

`continuous_simplified` uses an integer-grid approximation. Both modes apply the
same anniversary convention when $d = 0$ (the most common case). For $d > 0$,
the fractional terminal period in `continuous_simplified` uses $\lfloor t \rfloor$
rather than $\lfloor t - d \rfloor$.

## Net effective interest rate

When a benefit grows at rate $g$ and is discounted at rate $i$, a useful single-rate
equivalent is the **net effective interest rate**:

$$
i^* = \frac{1+i}{1+g} - 1
$$

When $i > g > 0$: $0 < i^* < i$ — payments grow but discounting dominates.
When $i = g$: $i^* = 0$ — growth and interest cancel; for a purely financial
annuity-due of term $n$ this gives $\ddot{a}_{\overline{n}|}^{(g,i)} = n$.
When $g > i$: $i^* < 0$ — growth exceeds discounting and the present value of
later payments exceeds that of earlier ones.

Lactuca does not use $i^*$ internally: all calculations operate directly on the
full sum with rate $i$ and growth factor $(1+g)^k$ per term. The net-rate identity
is therefore a useful consistency check or mental shorthand, not an implementation
detail. For growing annuities with mortality, the independence of survival
probabilities from $i^*$ means no further simplification arises beyond the annuity formula itself.

## Bibliographic reference

This convention is the standard in international actuarial literature:

- Bowers, N.L., Gerber, H.U., Hickman, J.C., Jones, D.A., Nesbitt, C.J.
  *Actuarial Mathematics*, 2nd ed. Society of Actuaries, 1997. §5.
- Dickson, D.C.M., Hardy, M.R., Waters, H.R.
  *Actuarial Mathematics for Life Contingent Risks*, 3rd ed. Cambridge University
  Press, 2020. §5.11.

## Piecewise growth rates and fractional `ts`

`GrowthRate` supports piecewise schedules of the form `rates=[r_0, r_1, r_2, ...]`
with associated `terms`. For **geometric** growth (`growth_type='g'`, the default),
when every segment has a one-year term (`terms=[1, 1, ...]`), the growth factor at
anniversary $k$ simplifies to:

$$F(k) = \prod_{i=0}^{k-1} (1 + r_i), \quad F(0) = 1$$

For multi-year segments (`terms=[T_0, T_1, ...]`), the same rate applies for $T_i$
consecutive anniversary years: within the segment that contains $k$, the factor
grows as $(1+r_s)^{k - C_s}$ where $C_s = \sum_{i<s} T_i$ is the cumulative start
of that segment. **Arithmetic** piecewise (`growth_type='a'`) is analogous, replacing
compound multiplication with linear accumulation. See {class}`lactuca.GrowthRate` for
the full specification and {doc}`growth_rates_guide` for worked examples of both types.

When a temporal shift `ts` is applied (e.g. `ax(x=60, ts=2.5)`), the engine calls
`GrowthRate.shifted(ts)` to discard the first `int(ts)` anniversary years of the
schedule before dispatching. This flooring is actuarially correct: benefit
revaluation (CPI indexation, salary scales, guaranteed annual increases) always
operates on complete policy years, not fractional ones.

### Example — piecewise schedule with fractional `ts`

```python
from lactuca import LifeTable, GrowthRate

gr = GrowthRate(rates=[0.01, 0.02, 0.05, 0.08], terms=[1, 1, 1])
# shifted(2.5) consumes years 0 and 1 (int(2.5) = 2)
print(gr.shifted(2.5))  # GrowthRate(rates=[0.05, 0.08], terms=[1], growth_type='g')

# At ts=2.5 the effective schedule starts at 0.05:
# F(0)=1, F(1)=1.05, F(2)=1.05×1.08, ...
lt = LifeTable("PASEM2020_Rel_1o", "m")
# A UserWarning is emitted because a GrowthRate is active with a fractional ts.
val = lt.ax(x=60, ts=2.5, m=12, ir=0.03, gr=gr)
print(round(val, 4))
```

## Policy on fractional `ts`

The `UserWarning` emitted on fractional `ts` fires **only when a `GrowthRate` is
active** (`gr != None`). Without a growth rate, fractional `ts` is silently accepted
regardless of `config.force_integer_ts`. This distinction is growth-specific: the
warning is meaningful only for escalating-benefit schedules where whole-year
anniversary indices matter.

For the full `force_integer_ts` reference — including the behaviour table and
interaction with `d` — see {doc}`prospective_reserve`.

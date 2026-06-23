# Irregular Cashflows

Most annuity formulas assume a **uniform payment schedule**: equal payments of $1/m$ per
period at regular $1/m$-year intervals throughout the term.  When a benefit design deviates
from this pattern — e.g., step-up pensions, inflation-indexed benefits, or arbitrary
single payment schedules — Lactuca's **irregular cashflow** interface provides a flexible
alternative.

:::{note}
`cashflow_times` (custom payment timing) is only supported by **immediate (postpayable)
annuity methods**: {meth}`lactuca.LifeTable.ax`, {meth}`lactuca.LifeTable.axy`,
{meth}`lactuca.LifeTable.axyz`, and {meth}`lactuca.LifeTable.ajoint`.

**Due (prepayable) variants** (`äx`, `äxy`, `äxyz`, `äjoint`) accept `cashflow_amounts`
for custom benefit scaling on the standard grid, but do **not** accept `cashflow_times`.
Payments always fall on the standard due grid $t_k = k/m$, $k = 0, 1, 2, \ldots$
See [Design note: why due annuities exclude `cashflow_times`](#due-cashflow-times-design-note).

To simulate due-style timing with explicit cashflows, use `ax` and include $t = 0$ as
the first element of `cashflow_times`; see [below](#due-via-explicit-times).
:::

## Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `cashflow_times` | sequence of `float` | Payment times in years from valuation date |
| `cashflow_amounts` | sequence of `float` | Payment amounts at each time |

Both arrays must have the same length.  **Times must be non-negative and strictly
ascending (no duplicates).**  **Amounts must be strictly positive (> 0).**

## How it works

Instead of internally generating the $t_k = k/m$ payment grid, the engine uses the
user-supplied `cashflow_times` array directly.  The present value calculation becomes:

$$\text{PV} = \sum_{k} c_k \cdot v^{t_k} \cdot {}_{t_k}p_x$$

where $c_k$ is `cashflow_amounts[k]`, $v^{t_k}$ is the discount factor from the
`InterestRate`, and ${}_{t_k}p_x$ is the survival probability to time $t_k$.

## Restrictions

| Condition | Behaviour |
|-----------|-----------|
| `calculation_mode != "discrete_precision"` | `ValueError` — `cashflow_times` requires `discrete_precision` |
| `m != 1` | `ValueError` — payment frequency must be 1 when explicit times are provided |
| `n > 0` | `ValueError` — `n` must be `None` (or 0) when explicit times are provided |
| `cashflow_amounts` in continuous modes | `ValueError` — not supported |
| Unsorted or duplicate times | `ValueError` |
| Any amount ≤ 0 | `ValueError` — amounts must be strictly positive |

:::{note}
`cashflow_amounts` can also be provided **without** `cashflow_times` to apply custom
per-payment amounts to a *regular* schedule (defined by `m` and `n`).  In that case
`m` can be any supported payment frequency (1, 2, 3, 4, 6, 12, 14, 24, 26, 52, or 365) and the array length must match the number of scheduled payments
$\lfloor n \cdot m \rfloor$.  Passing `cashflow_amounts` together with `gr=` raises a
`ValueError` — they are mutually exclusive; pass `gr=None` when using `cashflow_amounts`.
:::

## Use cases

### Step-up pension

Annual payments that step up by 10 % every five years over 20 years:

```python
from lactuca import LifeTable, payment_times, tiered_amounts

lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)

times   = payment_times(n=20, m=1)
amounts = tiered_amounts(
    times,
    breakpoints=[5, 10, 15],
    values=[1.00, 1.10, 1.21, 1.331],   # one value per tier
)

pv = lt.ax(65, cashflow_times=times, cashflow_amounts=amounts)
print(round(pv, 4))   # 14.5547  (unit benefit ≈ 1 per year; scale freely, e.g. × 10 000)
```

The `breakpoints` list is **inclusive on the right**: a payment at exactly $t = 5$
falls in the first tier and receives amount 1.00; a payment at $t = 5 + \epsilon$
falls in the second tier and receives 1.10.

### Inflation-indexed annuity

Monthly payments that grow at 2 % per year over 20 years:

```python
from lactuca import LifeTable, payment_times

lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)

times   = payment_times(n=20, m=12)        # t = 1/12, 2/12, ..., 20.0
amounts = (1.02 ** times) / 12.0           # 2 % annual inflation, 1/12 per month

pv = lt.ax(65, cashflow_times=times, cashflow_amounts=amounts)
print(round(pv, 3))   # 15.717
```

### Arbitrary benefit schedule

Any non-standard payment pattern — lump sums, variable benefits, or irregular timing:

```python
from lactuca import LifeTable

lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)

times   = [1.0, 2.0, 3.0, 5.0, 10.0]
amounts = [1.0, 1.0, 0.5, 2.0,  0.5]    # all strictly positive

pv = lt.ax(40, cashflow_times=times, cashflow_amounts=amounts)
print(round(pv, 4))   # 4.4566
```

(due-via-explicit-times)=
### Prepayable (due) payments via explicit times

The `äx` family does not accept explicit cashflow schedules, but a prepayable annuity —
where the first payment falls at $t = 0$ — can be constructed with `ax` by including
$t = 0$ in `cashflow_times`:

```python
import numpy as np
from lactuca import LifeTable

lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)
x, n = 65, 20

# Due schedule: payments at t = 0, 1, 2, ..., n-1
times_due   = np.arange(0, n, dtype=float)
amounts_due = np.ones(n)

pv_cashflow = lt.ax(x, cashflow_times=times_due, cashflow_amounts=amounts_due)
pv_due      = lt.äx(x, n=n)   # standard due annuity (prepayable)

print(np.isclose(pv_cashflow, pv_due))  # True
```

### Verifying equivalence with the standard formula

Explicit cashflows fully replicate a standard uniform annuity when the times and
amounts match the regular payment grid.  This confirms that both approaches produce
identical results (within floating-point tolerance):

```python
import numpy as np
from lactuca import LifeTable, payment_times

lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)
x, n, m = 65, 10, 4

# Standard quarterly annuity
pv_std = lt.ax(x, n=n, m=m)

# Equivalent via explicit cashflow_times + cashflow_amounts
times   = payment_times(n=n, m=m)                    # 0.25, 0.50, ..., 10.0
amounts = np.full(n * m, 1.0 / m)                    # 1/m per payment

pv_cf = lt.ax(x, cashflow_times=times, cashflow_amounts=amounts)

print(np.isclose(pv_std, pv_cf))  # True
```

(due-cashflow-times-design-note)=
## Design note: why due annuities exclude `cashflow_times`

Due annuities (`äx`, `äxy`, `äxyz`, `äjoint`) schedule payments at
$t_k = k/m$, $k = 0, 1, 2, \ldots$, beginning at $t = 0$.  This fixed grid is part
of the actuarial definition of a *prepayable* annuity-due.  Accepting an arbitrary
`cashflow_times` vector would allow the user to place payments at times that make the
contract neither due nor immediate — an actuarially ill-defined construct.

To keep the public API consistent with standard actuarial terminology:

- **`cashflow_amounts` is supported** for due annuities: it scales the benefit at each
  standard grid point without altering the timing.
- **`cashflow_times` is not supported**: any contract with non-standard payment timing
  should be modelled as a postpayable annuity (`ax` / `axy` / …).  When an immediate
  payment at $t = 0$ is needed, include it as the first element of `cashflow_times`
  (see [Prepayable payments via explicit times](#due-via-explicit-times)).

## Interaction with `m` and `n`

When `cashflow_times` is supplied, passing `m != 1` or `n > 0` raises a `ValueError`
immediately — these parameters are not silently ignored.  The payment schedule is
entirely determined by `cashflow_times`, so no frequency or duration hint is needed.

Use `cashflow_times=None` (the default) together with `m` and `n` for all standard
uniform-payment scenarios.

## Inspecting the cashflows

Pass `return_flows=True` to retrieve the per-payment arrays (times, discount factors,
survival probabilities, amounts, and present-value contributions) as a dict.
See {doc}`inspecting_cashflows` for the full key reference.

## Performance notes

For very long benefit schedules (e.g., 1 200 monthly payments), the engine is vectorised
over the full `cashflow_times` array using NumPy — no Python loop is executed.  This
applies to **single-life** batch (`ax`, `Ax`, …) as well as scalar calls whenever
`calculation_mode='discrete_precision'`.

In **multi-life** batch with `calculation_mode='discrete_precision'`, the same
vectorisation applies to products that accept `cashflow_times` (`axy`, `axyz`,
`ajoint`, `Axy`, `Axyz`, `Afirst`): one shared schedule is priced for *N* policies
in a single vectorised kernel, not *N* serial scalar calls.  Other calculation modes
still loop policy-by-policy.  Per-policy **different** schedules are not supported in
one call — group by product type as in {doc}`batch_calculations`.

## Custom per-payment amounts on a uniform schedule

When `cashflow_times` is *not* supplied, you can still pass `cashflow_amounts` to
override per-payment amounts on a **standard uniform grid** (defined by `m` and `n`).
The array length must equal $\lfloor n \cdot m \rfloor$.  All standard values of `m`
are allowed, and `n` must be positive.  Passing `cashflow_amounts` together with `gr=`
raises a `ValueError` — they are mutually exclusive; pass `gr=None` when using
`cashflow_amounts`.

This works for **both postpayable and due (prepayable)** annuities:
{meth}`~lactuca.LifeTable.ax`, {meth}`~lactuca.LifeTable.äx`,
{meth}`~lactuca.LifeTable.axy`, {meth}`~lactuca.LifeTable.äxy`, and so on.

This is useful when benefit amounts follow a deterministic but non-geometric pattern
that cannot be expressed as a constant `gr`:

```python
from lactuca import LifeTable

lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)

# Quarterly annuity for 5 years with arbitrary amounts (20 payments)
amounts = [1.0, 1.0, 1.1, 1.1,   # year 1
           1.1, 1.1, 1.2, 1.2,   # year 2
           1.2, 1.2, 1.3, 1.3,   # year 3
           1.3, 1.3, 1.4, 1.4,   # year 4
           1.4, 1.4, 1.5, 1.5]   # year 5

pv = lt.ax(60, n=5, m=4, cashflow_amounts=amounts)
print(round(pv, 4))   # 22.6638
```

The same interface works for due (prepayable) annuities.  The array must cover the
prepayable grid: $k = 0, 1, \ldots, \lfloor n \cdot m \rfloor - 1$,
so the length is still $\lfloor n \cdot m \rfloor$:

```python
from lactuca import LifeTable

lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)

# Quarterly due annuity for 5 years with step-up amounts (20 payments)
amounts = [1.0, 1.0, 1.1, 1.1,   # year 1 (payments at t = 0, 0.25, 0.5, 0.75)
           1.1, 1.1, 1.2, 1.2,   # year 2
           1.2, 1.2, 1.3, 1.3,   # year 3
           1.3, 1.3, 1.4, 1.4,   # year 4
           1.4, 1.4, 1.5, 1.5]   # year 5

pv = lt.äx(60, n=5, m=4, cashflow_amounts=amounts)
print(round(pv, 4))
```

## Generating payment schedules with `payment_times`

For common actuarial patterns — payments on specific months or quarters only —
the helper {func}`lactuca.payment_times` generates a ready-to-use
`cashflow_times` array without boilerplate:

```python
from lactuca import payment_times

# All quarterly payments for 10 years (default: all periods)
times = payment_times(n=10, m=4)
# times = [0.25, 0.50, 0.75, 1.00, ..., 10.00]  — 40 payments

# Quarterly payments in March (Q1) and September (Q3) only for 10 years
times = payment_times(n=10, m=4, selected_periods=[1, 3])
# times = [0.25, 0.75, 1.25, 1.75, ..., 9.25, 9.75]
```

| Argument | Type | Description |
|----------|------|-------------|
| `n` | `float` | Total duration in years (can be fractional) |
| `m` | `int` | Payment frequency: one of 1, 2, 3, 4, 6, 12, 14, 24, 26, 52, 365 |
| `selected_periods` | sequence of `int` or `None` | Period indices within each year, $1 \leq p \leq m$. `None` (default) selects all periods 1 to `m`. |

Payment times follow the formula $t = k + p/m$, where $k = 0, 1, \ldots$ are complete
years and $p$ is the selected period index.  The output is always sorted in ascending
order and free of duplicates.

### Supplemental payments added to a regular annuity

When a benefit design includes both a regular payment stream and additional supplemental
payments at specific times (such as an extra payment each mid-year and year-end), the
present value is simply the **sum of two separate calculations**: one for the regular
component using the standard formula, and one for the supplemental component using
explicit `cashflow_times`.

```python
import numpy as np
from lactuca import LifeTable
from lactuca import payment_times

lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)
x, n = 65, 20.0

# Regular component: standard monthly annuity
pv_regular = lt.ax(x, n=n, m=12)

# Supplemental component: two extra payments per year at months 6 and 12
t_extra  = payment_times(n=n, m=12, selected_periods=[6, 12])
a_extra  = np.full(t_extra.size, 1.0 / 12)   # each supplement equals one monthly amount
pv_extra = lt.ax(x, cashflow_times=t_extra, cashflow_amounts=a_extra)

# Total present value = regular + supplemental
pv_total = pv_regular + pv_extra

print(f"Regular  (12/yr):       {pv_regular:.4f}")
print(f"Supplemental (2/yr):    {pv_extra:.4f}")
print(f"Total    (14/yr):       {pv_total:.4f}")
print(f"Ratio total/regular:    {pv_total / pv_regular:.4f}")   # ≈ 14/12 ≈ 1.1667
```

The additivity of present values means there is no need to merge and sort the two
`cashflow_times` arrays — each component can be valued independently.

## Life insurances with irregular cashflows

The `cashflow_amounts` and `cashflow_times` parameters are also available on the
insurance functions {meth}`~lactuca.LifeTable.Ax`, {meth}`~lactuca.LifeTable.Axy`,
{meth}`~lactuca.LifeTable.Axyz`, and {meth}`~lactuca.LifeTable.Afirst`.  Both
parameters require ``calculation_mode='discrete_precision'``.

### Variable sum assured (`cashflow_amounts`)

When benefits escalate or vary by policy year — for example, a decreasing-term
insurance or a mortgage-linked policy — pass `cashflow_amounts` as a per-period array.

The present value formula with a variable sum assured $b_k$ is:

$$A = \sum_{k=1}^{n m} b_k \cdot v^{k/m} \cdot {}_{(k-1)/m}p_x \cdot q^{(k)}_x$$

where $q^{(k)}_x$ is the probability of dying in period $k$ and $b_k$ is the
corresponding benefit amount.

Restrictions mirror those for annuities:

| Condition | Behaviour |
|-----------|-----------|
| Length of `cashflow_amounts` ≠ $n \cdot m$ | `ValueError` |
| Any amount ≤ 0 | `ValueError` |
| `cashflow_amounts` with `gr` | `ValueError` — mutually exclusive |
| `calculation_mode != 'discrete_precision'` | `ValueError` |

**Example — growing sum assured (10-year term, annual payments):**

```python
from lactuca import LifeTable, payment_times

lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)

# Sum assured grows by 5 % per year over 10 years (capital creciente)
n, m = 10, 1
year_grid = payment_times(n, m)
amounts   = 1000.0 * (1.05 ** (year_grid - 1))   # [1000, 1050, 1102.5, ...]

pv = lt.Ax(50, n=n, m=m, cashflow_amounts=amounts)
print(round(pv, 4))   # present value of escalating death benefit
```

**Example — inspecting the per-period flows:**

```python
flows = lt.Ax(50, n=10, m=1, cashflow_amounts=amounts, return_flows=True)

# flows["amount"]        — benefit per period  (the supplied amounts)
# flows["present_value"] — PV contribution per period
print(flows["present_value"].sum())   # equals the scalar PV above
```

### Explicit payment schedule (`cashflow_times`)

For non-standard benefit structures — where the payment moment is not uniformly
spaced (e.g., a lump sum payable only at specific anniversaries) — pass
`cashflow_times` as an explicit array of payment moments.  The engine computes the
probability of death during each interval and discounts the benefit to the supplied
payment time.

Restrictions:

| Condition | Behaviour |
|-----------|-----------|
| `m != 1` when `cashflow_times` is provided | `ValueError` |
| `n > 0` when `cashflow_times` is provided | `ValueError` — use `n=None` |
| `cashflow_times` unsorted or with duplicates | `ValueError` |
| Any amount ≤ 0 | `ValueError` |
| `calculation_mode != 'discrete_precision'` | `ValueError` |

**Example — benefit payable only at years 3, 5, and 10:**

```python
from lactuca import LifeTable

lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)

# Death benefit of 100 000 paid at the end of year 3, 5, or 10
times   = [3.0, 5.0, 10.0]
amounts = [100_000.0, 100_000.0, 100_000.0]

pv = lt.Ax(40, cashflow_times=times, cashflow_amounts=amounts)
print(round(pv, 2))
```

### Interaction with `m` and mortality placement

When `cashflow_times` is provided to an insurance function, `m` controls the
granularity of the mortality interval — the sub-period width used to evaluate
conditional death probabilities.  It does **not** affect the payment timing, which
is set entirely by `cashflow_times`.

Pass `m=1` (the default) unless you need sub-annual mortality granularity.

## See also

- {func}`lactuca.payment_times` — helper for building payment schedules
- {doc}`last_payment_adjustment` — alignment of regular grids when $n$ is not a multiple of $1/m$
- {doc}`inspecting_cashflows` — `return_flows=True` key reference
- {doc}`notation_glossary` — $a$, $v$, ${}_{t}p_x$ definitions
- {doc}`calculation_modes` — discrete vs continuous modes

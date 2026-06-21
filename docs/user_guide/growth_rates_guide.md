# Growth Rates

This guide covers the `GrowthRate` class — how to create, configure, and use growth
rate objects in actuarial annuity and insurance calculations.

`GrowthRate` models a per-period revaluation factor applied to benefit amounts.
Applications are not limited to escalating benefits: common uses include pension CPI
indexation, salary-scale projections, guaranteed annual increases, and stress testing
with alternative growth assumptions. Growth can be **geometric** (compound, default)
or **arithmetic** (linear additive), with constant or piecewise schedules and
multi-scenario containers — analogous to `InterestRate`.

## Creating a GrowthRate

### Constant geometric growth

Geometric (compound) growth is the default: each payment receives a factor of
$(1+g)^k$, where $k$ is the number of complete policy years elapsed since the
first payment.

```python
from lactuca import GrowthRate

# 2% compound annual increase (growth_type='g' is the default)
gr = GrowthRate(0.02)
print(gr.summary())             # GrowthRate: geometric, constant rate = 0.020000
print(round(gr.factor(3), 6))   # 1.061208  — 1.02 ** 3
```

### Constant arithmetic growth

Arithmetic growth applies a flat linear increment: the factor at anniversary $k$
is $1 + g \cdot k$. Use this for benefit schedules expressed as a fixed percentage
of the original benefit (step increases), rather than a compound multiplier.

```python
from lactuca import GrowthRate

# 2% linear annual increase (additive)
gr_a = GrowthRate(0.02, growth_type='a')
print(gr_a.summary())      # GrowthRate: arithmetic, constant rate = 0.020000
print(gr_a.factor(3))      # 1.06  — 1 + 0.02 × 3
```

### Piecewise schedule

Pass `rates` and `terms` for a step-wise schedule. `rates` must have one more
element than `terms`; the last rate applies indefinitely:

```python
from lactuca import GrowthRate

# 1% for the first year, 2% thereafter
gr = GrowthRate(rates=[0.01, 0.02], terms=[1])
print(gr.summary())               # GrowthRate: geometric, piecewise terms=[1], rates=[0.01 0.02]
print(gr.factor(1))               # 1.01     — one period at 1%
print(round(gr.factor(2), 4))     # 1.0302   — 1.01 × 1.02
print(round(gr.factor(3), 6))     # 1.050804 — 1.01 × 1.02 ** 2
```

### When growth applies: `apply_from_first`

By default (`apply_from_first=False`) the first payment has factor 1 — growth
applies from the second payment onward. Set `apply_from_first=True` to apply
growth already to the first payment:

```python
from lactuca import GrowthRate

gr = GrowthRate(0.03, apply_from_first=False)   # standard convention
print(gr.factor(0))    # 1.0    — first payment, no growth yet
print(gr.factor(1))    # 1.03
print(gr.factor(2))    # 1.0609

gr_first = GrowthRate(0.03, apply_from_first=True)
print(gr_first.factor(0))    # 1.03   — growth applied even at t=0
print(gr_first.factor(1))    # 1.0609
```

### Multi-scenario container

Store named alternatives in a single object and switch with `active_scenario`:

```python
from lactuca import GrowthRate

gr = GrowthRate({
    "base":   GrowthRate(0.02),
    "stress": GrowthRate(0.04),
    "flat":   GrowthRate(0.00),
})
print(gr.summary())   # lists all scenarios and the active one

gr.active_scenario = "base"
print(round(gr.factor(5), 4))    # 1.1041  — 1.02 ** 5

gr.active_scenario = "stress"
print(round(gr.factor(5), 4))    # 1.2167  — 1.04 ** 5
```

## Computing growth factors

`GrowthRate.factor(t)` returns the cumulative growth factor at **anniversary index**
`t` (a non-negative integer). See [Anniversary convention](#anniversary-convention)
below for how the engine maps payment indices to anniversary indices.

```python
from lactuca import GrowthRate

gr = GrowthRate(0.03)

print(gr.factor(0))                      # 1.0     — reference payment (no growth yet)
print(gr.factor(1))                      # 1.03
print(round(gr.factor(5), 4))            # 1.1593  — 1.03 ** 5
print(gr.factor([0, 1, 2, 5]).round(4))  # [1.     1.03   1.0609 1.1593]
```

## Using with table methods

Pass `gr=` to `LifeTable` annuity and insurance methods. A plain `float` is also
accepted and wrapped automatically:

```python
from lactuca import LifeTable, GrowthRate

lt = LifeTable("PASEM2020_Rel_1o", "m")
gr = GrowthRate(0.02)

# Escalating whole-life annuity-due at 3% interest
lt.äx(65, ir=0.03, gr=gr)

# 20-year temporary annuity, monthly, using a plain float for growth
lt.äx(65, n=20, m=12, ir=0.03, gr=0.02)

# Escalating whole-life insurance
lt.Ax(65, ir=0.03, gr=gr)
```

The `gr=` parameter is accepted by the annuity methods `äx`, `ax` and their
joint-life variants (`äxy`, `axy`, `äxyz`, `axyz`), and by the insurance methods
`Ax`, `Axy`, `Axyz`. Pure endowments (`nEx`, `nExy`, `nExyz`) do not support `gr=`.

(anniversary-convention)=
## Anniversary convention

Growth factors are indexed by the number of **complete policy years** elapsed before
each payment. For payment frequency $m$, the $j$-th payment (0-based) receives
factor $F\!\left(\lfloor j/m \rfloor\right)$. The deferment parameter `d=` shifts
payment times but does **not** affect the anniversary index — growth always counts
complete years from the first payment of the stream, regardless of when it starts.

For the full mathematical derivation and worked examples, see {doc}`growth_conventions`.

## Piecewise schedules and `shifted()`

`GrowthRate.shifted(ts)` returns a new `GrowthRate` with the first `int(ts)`
anniversary years of the schedule consumed. The engine calls it automatically when
`ts > 0` so that the in-force growth schedule is correctly aligned from the
valuation date.

Only the **integer part** of `ts` advances the schedule: `shifted(2.5)` produces
the same result as `shifted(2)`. The fractional remainder does not move the growth
horizon — growth still revaluates at whole-year policy anniversaries counted from
contract inception. `config.force_integer_ts` (default `False`) controls whether
fractional `ts` values are accepted at the annuity level (with a `UserWarning`
reminding that the fractional part has no effect on the growth schedule) or raise
a `ValueError`. For full context and worked examples see {doc}`prospective_reserve`.

For the mathematical derivation of the schedule-shifting formula, see {doc}`growth_conventions`.

## Combining growth and interest

When both `gr=` and `ir=` are specified, the annuity engine compounds the growth
factor with the discount factor for each payment. For constant geometric growth, the
effective growth-adjusted discount factor per complete year simplifies to:

$$v_g = \frac{1 + g}{1 + i}$$

For piecewise or arithmetic schedules, and for sub-annual frequency $m > 1$, the
engine evaluates each payment individually. `GrowthRate` and `InterestRate` are
independent objects: neither modifies the other.

## Inspecting a GrowthRate

Use `print(gr)` or `gr.summary()` (alias for `str(gr)`) to get a human-readable
description of the current growth rate curve:

```python
from lactuca import GrowthRate

gr = GrowthRate(0.03)
print(gr)
# GrowthRate: geometric, constant rate = 0.030000

gr_neg = GrowthRate(-0.01)
print(gr_neg)
# GrowthRate: geometric, constant rate = -0.010000 (negative rate — shrinkage)

gr_pw = GrowthRate(rates=[0.02, 0.03], terms=[5])
print(gr_pw)
# GrowthRate: geometric, piecewise terms=[5], rates=[0.02 0.03]
```

Zero rates and negative rates are explicitly annotated so that they are visible in
IFRS 17 and Solvency II reserving contexts. Multi-scenario containers list all
scenario names alongside the active scenario.

Use `repr(gr)` for a compact, copy-pasteable constructor form (e.g. in logs or
debugging sessions).

## Mutating rates and terms

After construction, the rate values and segment durations of a piecewise schedule
can be updated in-place via the `rates` and `terms` setters. The array length (i.e.
number of segments) cannot be changed — create a new `GrowthRate` for structural
changes.

```python
from lactuca import GrowthRate

# Update a constant rate in-place
gr = GrowthRate(0.02)
gr.rates = [0.03]
print(round(gr.factor(5), 4))    # 1.1593  — 1.03 ** 5

# Update rates in a piecewise schedule
gr_pw = GrowthRate(rates=[0.01, 0.02], terms=[3])
gr_pw.rates = [0.025, 0.035]
print(gr_pw.factor(1))               # 1.025
print(round(gr_pw.factor(4), 6))     # 1.114582  — 1.025 ** 3 × 1.035

# Update segment durations (structure stays the same, only durations change)
gr_pw.terms = [5]   # segment now lasts 5 years instead of 3
print(gr_pw.get_segment_info(4)["segment_end"])    # 5
```

Setters validate the new values with the same rules as the constructor. Passing a
wrong-length array, a rate that violates the growth type constraint, or a non-positive
term raises `ValueError`.

Both setters are unavailable on multi-scenario containers — set individual scenario
instances directly. The `terms` setter is also unavailable on constant `GrowthRate`
instances (they have no segment structure).

### Copying to preserve the original

If you need to keep the original schedule intact while experimenting with a variant,
use `gr.copy()` instead of mutating in-place:

```python
from lactuca import GrowthRate

gr_base = GrowthRate(rates=[0.01, 0.02], terms=[5])

gr_stress = gr_base.copy()
gr_stress.rates = [0.02, 0.04]   # only gr_stress changes

print(gr_base.factor(3))    # 1.0303  (unchanged)
print(gr_stress.factor(3))  # 1.0612
```

`gr.copy()` produces an independent deep copy; arrays are not shared with the original.

## Querying rates and segment metadata

`get_rate(t)` retrieves the growth rate of the segment that applies at period `t`.
It accepts scalar or array input and follows the same scalar/array return convention
as `factor()`:

```python
from lactuca import GrowthRate

gr = GrowthRate(0.02)
print(gr.get_rate(0))              # 0.02
print(gr.get_rate([0, 5, 10]))     # [0.02 0.02 0.02]

gr_pw = GrowthRate(rates=[0.01, 0.02], terms=[3])
print(gr_pw.get_rate(2))               # 0.01  — period 2 is in segment 0
print(gr_pw.get_rate(3))               # 0.02  — period 3 starts the tail
print(gr_pw.get_rate([0, 2, 3, 10]))   # [0.01 0.01 0.02 0.02]
```

`get_segment_info(t)` returns a dict with full metadata about which segment applies
at period `t`:

```python
info = gr_pw.get_segment_info(1)
# {
#   'type': 'piecewise',
#   'rate': 0.01,
#   'growth_type': 'geometric',
#   'apply_from_first': False,
#   'segment_index': 0,
#   'segment_start': 0,
#   'segment_end': 3,
#   'is_terminal': False,
# }

tail_info = gr_pw.get_segment_info(100)
# segment_end is None for the open-ended tail
```

## Validation reports

`validate()` returns a structured dict describing the state of the growth rate
curve, including zero-rate and negative-rate warnings:

```python
from lactuca import GrowthRate

gr = GrowthRate(rates=[0.0, 0.02], terms=[3])
report = gr.validate()
# {
#   'type': 'piecewise',
#   'growth_type': 'geometric',
#   'apply_from_first': False,
#   'status': 'warnings',
#   'warnings': ['Zero growth rate found (1 occurrence(s)) — flat growth, no revaluation.'],
#   'rate_statistics': {'min': 0.0, 'max': 0.02, 'mean': 0.01,
#                       'zero_count': 1, 'negative_count': 0},
#   'segment_count': 1,
#   'total_defined_term': 3,
# }
```

Status is `'ok'` when no warnings are found, `'warnings'` otherwise. For
multi-scenario containers, the dict includes a `'scenarios'` sub-dict with one
entry per scenario.

## Curve analysis

`curve_analysis()` returns a quantitative summary of the growth curve, including
monotonicity flags and key cumulative factors at standard horizons:

```python
from lactuca import GrowthRate

gr = GrowthRate(rates=[0.01, 0.03, 0.02], terms=[3, 5])
ca = gr.curve_analysis()
# {
#   'type': 'piecewise',
#   'growth_type': 'geometric',
#   'rate_range': {'min': 0.01, 'max': 0.03},
#   'curve_properties': {
#       'is_flat': False,
#       'is_monotone_inc': False,
#       'is_monotone_dec': False,
#       'has_zero_rates': False,
#       'has_negative_rates': False,
#   },
#   'key_factors': {'t1': ..., 't5': ..., 't10': ..., 't20': ...},
#   'segment_count': 2,
#   'total_defined_term': 8,
# }
```

The `key_factors` sub-dict provides the cumulative `factor(t)` at `t = 1, 5, 10,
20`, giving a quick cross-section of the curve without enumerating the full
schedule.

## Exporting

`export()` serialises the growth rate configuration. In addition to the `'dict'`
and `'json'` formats, the `'regulatory'` format produces a JSON string enriched
with audit-trail metadata required for Solvency II and IFRS 17 reporting:

```python
from lactuca import GrowthRate
import json

gr = GrowthRate(0.02)

# Plain dict (default)
print(gr.export())

# JSON string
print(gr.export(format="json"))

# Regulatory JSON — adds export_timestamp (UTC ISO 8601), format_version, library
d = json.loads(gr.export(format="regulatory"))
print(d["export_timestamp"])   # e.g. "2025-07-15T10:32:11.123456+00:00"
print(d["format_version"])     # "1.0"
print(d["library"])            # "lactuca"
```

## Generating cashflow amounts

{meth}`GrowthRate.amounts` produces the escalated cashflow amount for each payment
in a schedule, using the anniversary convention described above.  It is the
companion to {func}`~lactuca.payment_times`: build the time grid first,
then derive the amounts vector.

```python
from lactuca import GrowthRate, payment_times
import numpy as np

gr = GrowthRate(0.02)              # 2 % geometric annual growth
times = payment_times(n=5, m=12)             # monthly payments for 5 years
amounts = gr.amounts(times, start=1_000.0, m=12)

# First year: 1 000 / month (no growth yet)
print(amounts[0])     # 1000.0
print(amounts[11])    # 1000.0
# Second year: × 1.02
print(round(amounts[12], 2))   # 1020.0
# Third year: × 1.02²
print(round(amounts[24], 2))   # 1040.4
```

For arithmetic growth, the amount increases by a fixed additive amount each year:

```python
gr_a = GrowthRate(0.05, growth_type='a')   # flat +5 % per year
times_a = payment_times(n=3, m=1)
print(gr_a.amounts(times_a, start=1000.0, m=1))
# [1000. 1050. 1100.]
```

For a piecewise schedule — for example, CPI-linked for 10 years, then a fixed rate:

```python
gr_pw = GrowthRate(rates=[0.03, 0.02], terms=[10])   # 3% for 10 yr, 2% thereafter
times_pw = gpt(n=15, m=1)
amounts_pw = gr_pw.amounts(times_pw, start=500.0, m=1)
print(round(amounts_pw[0], 2))    # 500.0  — year 1
print(round(amounts_pw[9], 2))    # 671.96 — year 10 (500 × 1.03^9)
print(round(amounts_pw[10], 2))   # 692.12 — year 11 (500 × 1.03^10)
print(round(amounts_pw[11], 2))   # 705.96 — year 12 (500 × 1.03^10 × 1.02)
```

:::{note}
`gr.amounts(times, start, m)` and `lt.äx(x, gr=gr)` use the **same** anniversary
index $\lfloor k/m \rfloor$.  Passing the `amounts` vector as `cashflow_amounts=`
to {meth}`~lactuca.LifeTable.ax` always produces the same present value as using
`gr=` in the formula — modulo rounding, because `cashflow_amounts` bypasses the
engine's internal growth loop and rounds at the input boundary.
:::

**Batch / DataFrame usage:** `gr=` accepts a Pandas or Polars `Series` directly.
A numeric `Series` is converted to per-policy floats.  An **object-dtype** `Series` of
`GrowthRate` instances (including piecewise curves) is treated identically to a list of
the same objects — no `.to_numpy()` is needed and no object flattening occurs.
Mixing `GrowthRate` objects and numeric values in a single `Series` raises `ValueError`.
See {doc}`batch_calculations` — *Broadcasting rules* for the complete parameter table.

## See also

- {doc}`growth_conventions` — full anniversary-convention reference and
  fractional-`ts` policy
- {doc}`interest_rates_guide` — analogous guide for `InterestRate`
- {doc}`calculation_modes` — how growth and interest interact across calculation
  modes
- {doc}`irregular_cashflows` — building non-standard cashflow schedules

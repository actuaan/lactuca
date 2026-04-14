# Utilities

Helper functions for actuarial calculations that are not tied to a specific table instance.

## Payment schedule helpers

{func}`~lactuca.generate_payment_times` generates the vector of payment times
(in years) for a given duration, payment frequency $m$, and optional subset of
periods within each year.  It is used internally by all annuity and insurance
calculation engines and is also available as a standalone tool for custom cashflow
construction.

Use this function directly when you need to inspect or override the default
payment grid that {class}`~lactuca.LifeTable` methods would produce for a given
$(n, m)$ combination, or when building {doc}`../user_guide/irregular_cashflows`.

```{seealso}
{doc}`../user_guide/irregular_cashflows` — Custom cashflow schedules for annuities and insurances.\
{doc}`../user_guide/last_payment_adjustment` — Fractional final payment handling.
```

```{eval-rst}
.. autofunction:: lactuca.generate_payment_times
```

## Tiered cashflow amounts

{func}`~lactuca.tiered_amounts` maps each payment time to a cashflow amount
according to a step-up / step-down schedule defined by breakpoints and values.
It is the recommended way to build piecewise-constant benefit schedules for use
with {func}`~lactuca.LifeTable.ax`.

```{seealso}
{doc}`../user_guide/irregular_cashflows` — Step-up pension example and further use cases.
```

```{eval-rst}
.. autofunction:: lactuca.tiered_amounts
```

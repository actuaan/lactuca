# GrowthRate

{class}`~lactuca.GrowthRate` models benefit revaluation schedules for life
annuities and insurances.  It supports:

- **Geometric** (compound) growth: $F(t) = \prod_{j=0}^{t-1}(1+g_j)$
- **Arithmetic** (linear additive) growth: $F(t) = 1 + \sum_{j=0}^{t-1} g_j$

where $F(0) = 1$ under the standard actuarial convention
(`apply_from_first=False`).

Like {class}`~lactuca.InterestRate`, growth rates can be **constant**,
**piecewise** (segment rates with durations), or **multi-scenario**.
{meth}`~lactuca.GrowthRate.shifted` generates time-shifted copies for
prospective reserve calculations.

```{seealso}
{doc}`../user_guide/growth_rates_guide` — Full guide to growth rate modeling.\
{doc}`../user_guide/growth_conventions` — Anniversary convention used by the annuity engine.
```

```{eval-rst}
.. autoclass:: lactuca.GrowthRate
   :members:
   :show-inheritance:
```

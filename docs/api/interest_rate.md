# InterestRate

{class}`~lactuca.InterestRate` represents a constant or piecewise term-structured
interest rate curve used for discounting actuarial cash-flows.  It supports:

- **Constant** rates: a single effective annual rate $i$.
- **Piecewise** curves: segment rates $i_1, i_2, \ldots$ with durations
  $t_1, t_2, \ldots$ (the last rate applies indefinitely).
- **Multi-scenario** containers: a named set of constant or piecewise curves
  with an active-scenario switch for stress testing.

Discount factors $v^n = (1+i)^{-n}$ are computed via {meth}`~lactuca.InterestRate.vn`
(direct $n$-year discount factor) and {meth}`~lactuca.InterestRate.vx`
(discount factor for $x - x_0$ years, equivalent to `vn(x - x0)`).  Term lengths
can be expressed in years, months, weeks, or days.

```{seealso}
{doc}`../user_guide/interest_rates_guide` — Full guide to interest rate curves and scenarios.
```

```{eval-rst}
.. autoclass:: lactuca.InterestRate
   :members:
   :show-inheritance:
```

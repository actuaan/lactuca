# Functional API

The functional API provides standalone functions that are **exactly equivalent**
to the corresponding {class}`~lactuca.LifeTable`, {class}`~lactuca.DisabilityTable`,
and {class}`~lactuca.ExitTable` OOP methods, with an explicit table argument replacing
the implicit ``self`` of the method call.  Multi-life functions (`äxy`, `äjoint`, etc.)
take a sequence of {class}`~lactuca.LifeTable` instances as their first argument.

Use the functional API when:

- You prefer a procedural style or want tabular functional composition.
- You are working with heterogeneous collections of table instances.
- You need to pass actuarial functions as callables to higher-order operations.

For all other workflows the OOP methods on {class}`~lactuca.LifeTable`,
{class}`~lactuca.DisabilityTable`, or {class}`~lactuca.ExitTable` are recommended.

```{seealso}
{doc}`../user_guide/functional_api` — Why and when to use the functional API.
```

## Single-life functions

```{eval-rst}
.. autofunction:: lactuca.äx
.. autofunction:: lactuca.ax
.. autofunction:: lactuca.Ax
.. autofunction:: lactuca.nEx
.. autofunction:: lactuca.px
.. autofunction:: lactuca.qx
.. autofunction:: lactuca.tpx
.. autofunction:: lactuca.tqx
.. autofunction:: lactuca.lx
.. autofunction:: lactuca.dx
.. autofunction:: lactuca.ex
.. autofunction:: lactuca.ex_continuous
.. autofunction:: lactuca.ix
.. autofunction:: lactuca.ox
```

## Commutation functions

```{eval-rst}
.. autofunction:: lactuca.Dx
.. autofunction:: lactuca.Nx
.. autofunction:: lactuca.Sx
.. autofunction:: lactuca.Cx
.. autofunction:: lactuca.Mx
.. autofunction:: lactuca.Rx
.. autofunction:: lactuca.Tx
.. autofunction:: lactuca.Tx_continuous
.. autofunction:: lactuca.Lx
.. autofunction:: lactuca.Lx_continuous
```

## Multi-life functions

```{eval-rst}
.. autofunction:: lactuca.äxy
.. autofunction:: lactuca.axy
.. autofunction:: lactuca.Axy
.. autofunction:: lactuca.nExy
.. autofunction:: lactuca.äjoint
.. autofunction:: lactuca.ajoint
.. autofunction:: lactuca.Afirst
.. autofunction:: lactuca.nEjoint
.. autofunction:: lactuca.äxyz
.. autofunction:: lactuca.axyz
.. autofunction:: lactuca.Axyz
.. autofunction:: lactuca.nExyz
```

:::{seealso}
{doc}`../user_guide/joint_life_calculations` — Full guide to joint-life and multi-life annuities and insurances.\
{doc}`utils` — {func}`~lactuca.generate_payment_times` and other helpers for
constructing custom payment grids (``cashflow_times`` parameter).
:::

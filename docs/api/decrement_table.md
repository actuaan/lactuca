# DecrementTable

{class}`~lactuca.tables.DecrementTable` is the abstract base class for all
single-decrement table types ({class}`~lactuca.LifeTable`,
{class}`~lactuca.DisabilityTable`, {class}`~lactuca.ExitTable`).  It provides
the shared logic for table construction, the survival function $l_x$ with
radix $l_0$, the governing decrement probability ($q_x$, $i_x$, or $o_x$
depending on the concrete subclass), and decrement modification.
{class}`~lactuca.LifeTable` extends it further with the full suite of
annuity, insurance, commutation, and life expectancy calculations.

Direct instantiation of `DecrementTable` is not supported — use one of
the concrete subclasses.

```{seealso}
{doc}`../user_guide/tables_taxonomy` — Overview of table types and decrement conventions.\
{doc}`../user_guide/using_tables` — Loading, inspecting, and modifying tables.\
{doc}`../user_guide/modifying_decrements` — Scaling and shocking decrement rates.
```

```{eval-rst}
.. autoclass:: lactuca.tables.DecrementTable
   :members:
   :show-inheritance:
```

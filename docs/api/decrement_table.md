# DecrementTable

{class}`~lactuca.tables.DecrementTable` is the abstract base class for all
single-decrement table types ({class}`~lactuca.LifeTable`,
{class}`~lactuca.DisabilityTable`, {class}`~lactuca.ExitTable`).  It provides
the shared logic for table construction, the survival function $l_x$ with
radix $l_0 = 1{,}000{,}000$, the governing decrement probability ($q_x$, $i_x$, or $o_x$
depending on the concrete subclass), and decrement modification
({meth}`~lactuca.tables.DecrementTable.modify_qx` on life tables;
{meth}`~lactuca.DisabilityTable.modify_ix` and {meth}`~lactuca.ExitTable.modify_ox`
on the other subclasses).
{class}`~lactuca.LifeTable` extends it further with the full suite of
annuity, insurance, commutation, and life expectancy calculations.

Direct instantiation of `DecrementTable` is not supported — use one of
the concrete subclasses.  Import the base class as
`from lactuca.tables import DecrementTable` (it is not re-exported from the
top-level `lactuca` package).

```{seealso}
{doc}`../user_guide/tables_taxonomy` — Overview of table types and decrement conventions.\
{doc}`../user_guide/using_tables` — Loading, inspecting, and modifying tables.\
{doc}`../user_guide/modifying_decrements` — Scaling, aggravated risk, and `table_combination`.\
{doc}`life_table` — {class}`~lactuca.LifeTable` reference.\
{doc}`disability_table` — {class}`~lactuca.DisabilityTable` reference.\
{doc}`exit_table` — {class}`~lactuca.ExitTable` reference.
```

```{eval-rst}
.. autoclass:: lactuca.tables.DecrementTable
   :members:
   :show-inheritance:
```

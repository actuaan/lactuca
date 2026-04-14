# ExitTable

{class}`~lactuca.ExitTable` extends {class}`~lactuca.tables.DecrementTable`
for **lapse / withdrawal** tables.  The governing decrement is the exit rate
$o_x$, which represents the probability that a contract or member aged $x$
lapses, surrenders, or withdraws within one year.

Exit tables are used in persistency analysis, lapse-risk pricing, and
multi-decrement models for group pension and collective insurance products.
Table files carry $o_x$ columns (prefixed `ox_m`, `ox_f`, or `ox_u`) and may include
generational improvement factors for cohort-based exit rates.

```{seealso}
{doc}`../user_guide/tables_taxonomy` — Overview of all table types and decrement conventions.\
{doc}`../user_guide/using_tables` — Loading and inspecting tables.\
{doc}`../user_guide/modifying_decrements` — Scaling and shocking decrement rates.\
{doc}`../user_guide/mortality_improvement` — Generational tables and improvement factors.
```

```{eval-rst}
.. autoclass:: lactuca.ExitTable
   :members:
   :show-inheritance:
```

## ExitTable-specific members

These members are defined on {class}`~lactuca.ExitTable` itself and are
not present on {class}`~lactuca.tables.DecrementTable`.

### Actuarial methods

```{eval-rst}
.. autosummary::
   :nosignatures:

   ~lactuca.ExitTable.ox
```

### Modification

```{eval-rst}
.. autosummary::
   :nosignatures:

   ~lactuca.ExitTable.modify_ox
```

## Inherited from DecrementTable

The following members are inherited from {class}`~lactuca.tables.DecrementTable`.
See the {doc}`DecrementTable reference <decrement_table>` for full documentation
of each member.

:::{note}
`qx` is not available on `ExitTable` — the primary decrement is `ox`
(exit / lapse rate). Calling `qx` raises `NotImplementedError`.
:::

### Actuarial methods

```{eval-rst}
.. autosummary::
   :nosignatures:

   ~lactuca.tables.DecrementTable.lx
   ~lactuca.tables.DecrementTable.px
   ~lactuca.tables.DecrementTable.tpx
   ~lactuca.tables.DecrementTable.tqx
   ~lactuca.tables.DecrementTable.dx
```

### Modification

```{eval-rst}
.. autosummary::
   :nosignatures:

   ~lactuca.tables.DecrementTable.reset_modifications
   ~lactuca.tables.DecrementTable.copy
```

### Display

```{eval-rst}
.. autosummary::
   :nosignatures:

   ~lactuca.tables.DecrementTable.view_data
   ~lactuca.tables.DecrementTable.head
   ~lactuca.tables.DecrementTable.tail
```

### State

```{eval-rst}
.. autosummary::
   :nosignatures:

   ~lactuca.tables.DecrementTable.sex
   ~lactuca.tables.DecrementTable.cohort
   ~lactuca.tables.DecrementTable.duration
   ~lactuca.tables.DecrementTable.unisex_blend
   ~lactuca.tables.DecrementTable.w
   ~lactuca.tables.DecrementTable.modified
   ~lactuca.tables.DecrementTable.modifications_applied
   ~lactuca.tables.DecrementTable.is_select
```

### Table metadata

```{eval-rst}
.. autosummary::
   :nosignatures:

   ~lactuca.tables.DecrementTable.table_name
   ~lactuca.tables.DecrementTable.table
   ~lactuca.tables.DecrementTable.generational
   ~lactuca.tables.DecrementTable.base_year
   ~lactuca.tables.DecrementTable.omega
   ~lactuca.tables.DecrementTable.start_age
   ~lactuca.tables.DecrementTable.description
   ~lactuca.tables.DecrementTable.valid_sexes
   ~lactuca.tables.DecrementTable.sex_independent
   ~lactuca.tables.DecrementTable.generational_formula_type
   ~lactuca.tables.DecrementTable.select
   ~lactuca.tables.DecrementTable.select_period
   ~lactuca.tables.DecrementTable.start_duration
   ~lactuca.tables.DecrementTable.mi_by_duration
   ~lactuca.tables.DecrementTable.mi_structure
   ~lactuca.tables.DecrementTable.grid_years
   ~lactuca.tables.DecrementTable.file_name
   ~lactuca.tables.DecrementTable.file_path
   ~lactuca.tables.DecrementTable.data
   ~lactuca.tables.DecrementTable.metadata
```

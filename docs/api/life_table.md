# LifeTable

{class}`~lactuca.LifeTable` is the core actuarial table class in Lactuca.
It inherits from {class}`~lactuca.tables.DecrementTable` and adds the full
set of life annuity ($\ddot{a}_x$, $a_x$, $\bar{a}_x$), life insurance ($A_x$),
pure endowment (${}_{n}E_x$), commutation function ($D_x$, $N_x$, $M_x$, …),
and life expectancy ($\mathring{e}_x$) methods.

A life table is identified by a table name (`.ltk` file), a sex code
(`'m'`, `'f'`, or `'u'`), and optionally a birth-year cohort for
generational tables that include mortality improvement factors.

```{seealso}
{doc}`../user_guide/getting_started` — Quick-start guide with common patterns.\
{doc}`../user_guide/life_annuities_guide` — Full annuity calculation reference.\
{doc}`../user_guide/life_insurances_guide` — Insurance and endowment reference.\
{doc}`../user_guide/commutation_functions` — Commutation functions reference.\
{doc}`../user_guide/calculation_modes` — Discrete vs. continuous modes.\
{doc}`../user_guide/joint_life_calculations` — Joint-life methods (äxy, axy, Axy, …).\
{doc}`../user_guide/lx_interpolation` — Fractional-age survival assumptions (UDD vs. CFM).\
{doc}`../user_guide/deferment` — Deferred annuities and the ``d=`` parameter.\
{doc}`../user_guide/mortality_improvement` — Generational tables and improvement factors.
```

```{eval-rst}
.. autoclass:: lactuca.LifeTable
   :members:
   :show-inheritance:
```

## Inherited from DecrementTable

The following members are inherited from {class}`~lactuca.tables.DecrementTable`
without modification. See the {doc}`DecrementTable reference <decrement_table>`
for full documentation of each member.

### Actuarial methods

```{eval-rst}
.. autosummary::
   :nosignatures:

   ~lactuca.tables.DecrementTable.lx
   ~lactuca.tables.DecrementTable.px
   ~lactuca.tables.DecrementTable.qx
   ~lactuca.tables.DecrementTable.tpx
   ~lactuca.tables.DecrementTable.tqx
   ~lactuca.tables.DecrementTable.dx
```

### Modification

```{eval-rst}
.. autosummary::
   :nosignatures:

   ~lactuca.tables.DecrementTable.modify_qx
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

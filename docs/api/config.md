# Config

{class}`~lactuca.Config` is the global configuration singleton that controls all
precision, calculation, and table-loading settings for a Lactuca session.
Changes take effect immediately for all subsequent calculations in the same
Python process; call {meth}`~lactuca.Config.reset` to restore factory defaults.

Key setting groups:

| Group | Settings | Guide |
|-------|----------|-------|
| **Calculation mode** | `calculation_mode` | {doc}`../user_guide/calculation_modes` |
| **Decimal precision** | `decimals.*` | {doc}`../user_guide/decimals_rounding` |
| **$l_x$ interpolation** | `lx_interpolation` (`"linear"` / `"exponential"`) | {doc}`../user_guide/lx_interpolation` |
| **Mortality placement** | `mortality_placement` (`"beginning"` / `"mid"` / `"end"`) | {doc}`../user_guide/life_insurances_guide` |
| **Force of mortality** | `force_mortality_method` | {doc}`../user_guide/force_mortality_methods` |
| **Calendar constants** | `days_per_year`, `weeks_per_year` | {doc}`../user_guide/numerical_precision` |
| **Date parsing** | `date_format` | {doc}`../user_guide/dates_guide` |
| **Tables path** | `tables_path` | {doc}`../user_guide/using_tables` |
| **Config file path** | `config_path` (alias: `path`) | {doc}`../user_guide/configuration` |
| **Force integer ts** | `force_integer_ts` | {doc}`../user_guide/last_payment_adjustment` |

```{seealso}
{doc}`../user_guide/configuration` — Narrative guide to all Config settings with worked examples.
```

```{eval-rst}
.. autoclass:: lactuca.Config
   :members:
   :show-inheritance:
```

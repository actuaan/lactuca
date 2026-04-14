# TableBuilder

{class}`~lactuca.TableBuilder` provides a fluent interface for constructing custom
actuarial tables programmatically from raw decrement arrays and saving them as
`.ltk` files with automatic metadata validation and SHA-256 integrity hashing.

Supported decrement types:

| Array | Symbol | Table type |
|-------|--------|------------|
| `qx_m` / `qx_f` / `qx_u` | $q_x$ | Mortality ({class}`~lactuca.LifeTable`) |
| `ix_m` / `ix_f` / `ix_u` | $i_x$ | Disability inception rate ({class}`~lactuca.DisabilityTable`) |
| `ox_m` / `ox_f` / `ox_u` | $o_x$ | Exit / lapse rate ({class}`~lactuca.ExitTable`) |

Generational tables additionally require improvement rate columns (`mi_m`, `mi_f`, and/or `mi_u`).

```{seealso}
{doc}`../user_guide/building_tables` — Step-by-step guide to building custom tables.\
{doc}`../user_guide/tables_taxonomy` — Overview of table types and column conventions.
```

```{eval-rst}
.. autoclass:: lactuca.TableBuilder
   :members:
   :show-inheritance:
```

## `read_table`

Lightweight function for loading a `.ltk` file without building a full `TableBuilder`
instance.  Returns a `(pl.DataFrame, dict)` pair — a Polars DataFrame with the table
data and a `dict` of normalised metadata — and supports both strict integrity-checking
mode and best-effort recovery mode.

```{eval-rst}
.. autofunction:: lactuca.read_table
```

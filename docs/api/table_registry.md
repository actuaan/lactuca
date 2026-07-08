# Table registry utilities

{class}`~lactuca.TableRegistry` and {func}`~lactuca.configure_all` support
**deferred construction** workflows: building table shells before metadata is known,
and caching fully configured instances for heterogeneous batch calculations.

Neither is required for batch — they are convenience utilities for building a
**list of configured** table instances without aliasing or redundant construction.

For narrative examples, see {ref}`deferred-construction` and {ref}`tableregistry` in
{doc}`../user_guide/using_tables`.  Recipe 18 in {doc}`../cookbook` shows a full
heterogeneous portfolio with the functional batch API.

## When to use which tool

| Situation | Recommended approach | Code example |
|---|---|---|
| Same cohort (or duration) processed **one group at a time** | `pending=True` + {meth}`~lactuca.tables.DecrementTable.configure` per group | {ref}`recipe 17 <recipe-17>` ({doc}`../cookbook`) |
| Vectorial zip on sex with a **shared deferred** cohort | `pending=True` + {func}`~lactuca.configure_all` | {ref}`deferred-construction` § `configure_all()` ({doc}`../user_guide/using_tables`) |
| Mixed demographics in **one batch**, keys built incrementally | {class}`~lactuca.TableRegistry` + {meth}`~lactuca.TableRegistry.get_or_create` | {ref}`recipe 18 <recipe-18>` ({doc}`../cookbook`) |
| Large portfolio; all unique `(sex, cohort)` pairs **known upfront** | `return_dict=True` + {class}`~lactuca.TableKey` lookup | {ref}`lookup dict <cohort-lookup-dict>` ({doc}`../user_guide/batch_calculations`) |
| Study grid / cartesian parameter sweep | `return_dict=True` + `cartesian=True` | {ref}`return_dict lookup <return-dict-lookup>` ({doc}`../user_guide/batch_calculations`) |
| Few distinct cohorts; assemble list by hand | Vectorial zip constructor | {ref}`few distinct cohorts <cohort-few-distinct>` ({doc}`../user_guide/batch_calculations`) |
| Very large portfolio; **memory** constrained | `groupby` + one instance + setters | {ref}`memory-optimal <cohort-memory-optimal>` ({doc}`../user_guide/batch_calculations`) |
| Small portfolio; metadata known per row | Direct constructor in a list comp | ``[LifeTable(..., cohort=p["cohort"]) for p in policies]`` |
| Select table; **different duration** per policy | {class}`~lactuca.TableRegistry` (`duration` in cache key) | {ref}`tableregistry` ({doc}`../user_guide/using_tables`); {ref}`recipe 18 <recipe-18>` |
| Same demographic key; **different** interest rate | Pass `ir=` to the batch method | {ref}`multi-table batch <multi-table-batch-functional-api>` ({doc}`../user_guide/batch_calculations`) |

:::{tip}
{class}`~lactuca.TableRegistry` is a convenience wrapper around the same idea as the
{ref}`lookup dict <cohort-lookup-dict>` pattern — lazy {meth}`~lactuca.TableRegistry.get_or_create`
with LRU instead of building every unique key in one constructor call.
:::

```{seealso}
{doc}`../user_guide/batch_calculations` — Pending tables in batch; cohort/duration portfolio patterns.\\
{doc}`decrement_table` — Base class {meth}`~lactuca.tables.DecrementTable.configure` and {meth}`~lactuca.tables.DecrementTable.batch_update`.
```

## TableRegistry

```{eval-rst}
.. autoclass:: lactuca.TableRegistry
   :members:
```

## configure_all

```{eval-rst}
.. autofunction:: lactuca.configure_all
```

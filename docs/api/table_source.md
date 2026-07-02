# TableSource

{class}`~lactuca.TableSource` loads an `.ltk` actuarial table file from disk and
exposes the raw decrement arrays and metadata for use by {class}`~lactuca.LifeTable`,
{class}`~lactuca.DisabilityTable`, and {class}`~lactuca.ExitTable` constructors
(each constructor delegates to `TableSource` internally).

Table files are located in the directory configured by
{attr}`~lactuca.Config.tables_path` (default: absolute path to `actuarial_tables/`
in the working directory at the time of the first `Config()` call).  A disk-read
cache keyed on file path and modification time avoids redundant I/O for repeated
access to the same table within a session.

```{note}
`TableSource` is primarily an internal component.  Users normally access table
data through {class}`~lactuca.LifeTable` and its siblings rather than constructing
a `TableSource` directly.

Bundled catalogue tables are shipped in-repo as Python payloads; install them to
``tables_path`` as ``.ltk`` files with ``Tables.install()`` (import
``Tables`` from ``lactuca.tables.data``) before first use — see
{doc}`../user_guide/bundled_tables`.
```

```{seealso}
{doc}`../user_guide/using_tables` — Loading and inspecting tables.\
{doc}`../user_guide/bundled_tables` — Tables bundled with Lactuca.
```

```{eval-rst}
.. autoclass:: lactuca.TableSource
   :members:
   :show-inheritance:
```

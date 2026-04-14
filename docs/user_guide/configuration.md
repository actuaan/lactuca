# Configuration

This page explains, in usage order, the full public functionality of Lactuca's global configuration object:

- The class `Config`
- The top-level alias `config`

Both refer to the same process-level singleton.

## Mental model

Think of configuration in three layers:

1. **Current in-memory state** (what calculations use immediately).
2. **Persistent TOML file** (optional on-disk state).
3. **Singleton lifecycle** (one shared object per Python process).

Most user confusion disappears if you keep these three layers separate.

## Accessing the singleton

Use either import style:

```python
from lactuca import Config, config

cfg1 = Config()
cfg2 = Config()

print(cfg1 is cfg2)     # True
print(cfg1 is config)   # True
```

:::{note}
Any change to `Config`/`config` applies immediately to all subsequent
calculations in the same Python process.
:::

## Reading configuration values

### Read one key: `get`

```python
from lactuca import Config

cfg = Config()
mode = cfg.get("calculation_mode")
print(mode)  # 'discrete_precision'
```

If the key does not exist and no fallback is provided, `get` raises `KeyError`.

```python
from lactuca import Config

cfg = Config()
value = cfg.get("non_existing_key", "fallback")
print(value)  # 'fallback'
```

### Read all keys: `as_dict`

```python
from lactuca import Config

cfg = Config()
snapshot = cfg.as_dict()
print("calculation_mode" in snapshot)  # True
```

`as_dict()` returns a shallow snapshot of the current validated state.

### Iterate key/value pairs

`Config` is iterable:

```python
from lactuca import Config

cfg = Config()
for key, value in cfg:
    if key in {"calculation_mode", "lx_interpolation"}:
        print(key, value)
```

## Updating values

### Update one key: `set`

```python
from lactuca import config

config.set("calculation_mode", "continuous_precision")
print(config.calculation_mode)  # 'continuous_precision'

config.reset_to_defaults()  # restore defaults after example
```

### Update many keys atomically: `set_many`

```python
from lactuca import config

config.set_many({
    "calculation_mode": "discrete_simplified",
    "lx_interpolation": "exponential",
    "decimals_annuities": 8,
})

print(config.calculation_mode)      # 'discrete_simplified'
print(config.lx_interpolation)      # 'exponential'
print(config.decimals.annuities)    # 8

config.reset_to_defaults()  # restore defaults after example
```

`set_many` is all-or-nothing: if one value is invalid, none of the changes are applied.
It is also thread-safe; concurrent calls from multiple threads are serialized automatically.

### Property style (shortcut)

For common settings, properties are available:

```python
from lactuca import config

config.calculation_mode = "discrete_precision"
config.lx_interpolation = "linear"
config.force_integer_ts = False

config.reset_to_defaults()  # restore defaults after example
```

## Decimal settings via `decimals` proxy

Use the grouped proxy for autocompletion and readability:

```python
from lactuca import config

config.decimals.qx = 8
config.decimals.lx = 12
config.decimals.annuities = 8

print(config.decimals.qx)         # 8
print(config.decimals.annuities)  # 8

config.reset_to_defaults()  # restore defaults after example
```

Equivalent flat-key form:

```python
from lactuca import config

config.set("decimals_qx", 8)
print(config.decimals.qx)  # 8

config.reset_to_defaults()  # restore defaults after example
```

For full rounding semantics and table-specific availability, see {doc}`decimals_rounding`.

## Persistence methods (TOML)

### `config_path` and `path`

Both properties expose the current target file path:

```python
from lactuca import Config

cfg = Config()
print(cfg.config_path)
print(cfg.path)  # alias of config_path
```

### `save(file_path=None, overwrite=False)`

- Serializes current in-memory config to TOML.
- Writes atomically.
- Raises `FileExistsError` if target exists with different content and `overwrite=False`.

```python
from lactuca import config

config.set("calculation_mode", "continuous_precision")
config.save("./my_config.toml", overwrite=True)

config.reset_to_defaults()  # restore defaults after example
```

### `save_as(file_path, overwrite=False)`

Convenience wrapper around `save` for explicit "save to new path" intent.

```python
from lactuca import config

config.save_as("./backup_config.toml", overwrite=True)
```

### `save_if_changed(file_path=None)`

Writes only when on-disk content differs. Returns `True` if a write occurred, `False` otherwise.

```python
from lactuca import config

changed = config.save_if_changed("./my_config.toml")
print(changed)  # True or False
```

### `load(file_path=None)`

Loads TOML values and merges them over the current in-memory state.
Only keys present in the file are overwritten.

```python
from lactuca import config

config.load("./my_config.toml")
print(config.calculation_mode)
```

:::{note}
`Config(config_path="...")` is honored only on first singleton creation.
If the singleton already exists, use `config.load("...")` to switch to another file explicitly.
:::

## Reset methods and singleton lifecycle

### `reset_to_defaults()` (instance method)

- Restores factory defaults in memory.
- Keeps the same singleton object identity.
- Does not write to disk automatically.

```python
from lactuca import config

config.reset_to_defaults()
```

### `Config.reset()` (class method)

- Clears the singleton instance.
- Next `Config()` call creates a fresh object.
- Existing old references become stale.

```python
from lactuca import Config

old_cfg = Config()
Config.reset()
new_cfg = Config()

print(old_cfg is new_cfg)  # False
```

Use `Config.reset()` mainly in tests and controlled bootstrap scenarios.
For notebooks and scripts, `reset_to_defaults()` is usually safer.

## Recommended usage patterns

### Pattern A: Notebook/session work

```python
from lactuca import config

config.reset_to_defaults()
config.calculation_mode = "discrete_precision"
config.lx_interpolation = "linear"
```

### Pattern B: Batch update + persist once

```python
from lactuca import config

config.set_many({
    "calculation_mode": "continuous_precision",
    "lx_interpolation": "exponential",
    "decimals_annuities": 10,
})
config.save_if_changed("./pricing_config.toml")

config.reset_to_defaults()  # optional cleanup for shared sessions
```

### Pattern C: Test isolation

```python
from lactuca import Config

Config.reset()          # ensure fresh singleton
cfg = Config()
cfg.reset_to_defaults() # deterministic baseline
```

## Settings reference

### Table path

| Setting | Default | Allowed values / type | Purpose |
|---------|---------|------------------------|---------|
| `tables_path` | `actuarial_tables/` (resolved from CWD) | path-like / string | Directory for `.ltk` table read/write operations. |

See {doc}`bundled_tables` for installation and usage of bundled tables.

### Calculation controls

| Setting | Default | Allowed values | Purpose |
|---------|---------|----------------|---------|
| `calculation_mode` | `"discrete_precision"` | `"discrete_precision"`, `"discrete_simplified"`, `"continuous_precision"`, `"continuous_simplified"` | Selects valuation engine behavior for annuities, insurances, and pure endowments. |
| `lx_interpolation` | `"linear"` | `"linear"`, `"exponential"` | Fractional-age survival hypothesis: UDD (`linear`) or constant force (`exponential`). |
| `force_mortality_method` | `"finite_difference"` | `"finite_difference"`, `"spline"`, `"kernel"` | Method used by force-of-mortality routines. |
| `mortality_placement` | `"mid"` | `"beginning"`, `"mid"`, `"end"` | Timing offset for death-benefit discounting in insurance calculations. |
| `force_integer_ts` | `False` | `bool` | If `True`, rejects fractional `ts` shifts. |

See:

- {doc}`calculation_modes`
- {doc}`lx_interpolation`
- {doc}`force_mortality_methods`

### Calendar and date parsing

| Setting | Default | Allowed values | Purpose |
|---------|---------|----------------|---------|
| `date_format` | `"ymd"` | `"ymd"`, `"dmy"`, `"mdy"`, `"ymd_int"` | Parsing order for ambiguous date strings and default format for date formatting helpers. |
| `days_per_year` | `365.25` | `360`, `365`, `365.25`, `365.2425`, `366` | Calendar constant for date-based age/fraction conversions. |
| `weeks_per_year` | `52.1775` | `52`, `52.1429`, `52.1775` | Weekly conversion constant. |

`"ymd_int"` accepts 8-digit `YYYYMMDD` input as integer (example: `19650315`) or numeric string (example: `"19650315"`).

See {doc}`dates_guide` for all accepted date input forms.

### Decimal precision

All decimal settings accept non-negative integers.

| Setting | Default |
|---------|---------|
| `decimals_lx` | 15 |
| `decimals_dx` | 15 |
| `decimals_qx` | 15 |
| `decimals_px` | 15 |
| `decimals_tpx` | 15 |
| `decimals_tqx` | 15 |
| `decimals_ix` | 15 |
| `decimals_ox` | 15 |
| `decimals_ex` | 15 |
| `decimals_annuities` | 15 |
| `decimals_insurances` | 15 |
| `decimals_Lx` | 10 |
| `decimals_Tx` | 10 |
| `decimals_Dx` | 10 |
| `decimals_Nx` | 10 |
| `decimals_Sx` | 10 |
| `decimals_Cx` | 10 |
| `decimals_Mx` | 10 |
| `decimals_Rx` | 10 |

For rounding pipeline details and per-table availability, see {doc}`decimals_rounding`.

## TOML file layout

`Config` recognizes a canonical **section layout** for TOML files:

- `[paths]`
- `[calculation]`
- `[calendar]`
- `[mortality]`
- `[decimals]`

In this context, *canonical* means that these are the standard section names
used by Lactuca when reading and writing TOML configuration files.
It does **not** mean that the example below is an exhaustive list of every
supported key.

The following example is intentionally abbreviated to show the expected
structure. In particular, the `[decimals]` section may contain all supported
decimal keys, not just the four shown below.

```toml
[paths]
actuarial_tables = "/data/my_actuarial_tables"

[calculation]
calculation_mode = "discrete_precision"
lx_interpolation = "linear"
force_integer_ts = false

[calendar]
date_format = "ymd"
days_per_year = 365.25
weeks_per_year = 52.1775

[mortality]
force_mortality_method = "finite_difference"
mortality_placement = "mid"

[decimals]
lx = 15
qx = 8
annuities = 8
insurances = 8
```

When you call `load()`, omitted keys are left unchanged if a configuration is
already loaded, or keep their library defaults if they were never overridden.

When you call `save()` or `save_if_changed()`, Lactuca writes the full current
configuration using the canonical section layout, including all decimal fields
present in `Config`.

## See also

- {doc}`calculation_modes` — engine behavior by calculation mode
- {doc}`lx_interpolation` — UDD vs CFM interpolation assumptions
- {doc}`force_mortality_methods` — force-of-mortality method comparison
- {doc}`decimals_rounding` — decimal write semantics and rounding pipeline
- {doc}`dates_guide` — accepted date formats and date parsing behavior
- {doc}`bundled_tables` — bundled table installation and paths

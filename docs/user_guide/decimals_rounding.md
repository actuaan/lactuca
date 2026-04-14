# Decimal Precision and Rounding

Lactuca provides fine-grained control over the number of decimal places reported by every
public method.  All settings live in the global `Config` singleton, accessible via
`Config()` (the class constructor always returns the same instance) or the pre-created
module-level alias `config` — both exported from the package.  Settings are also surfaced
as read-only properties on table instances through the `decimals` attribute.

## Default precision strategy

The defaults reflect a deliberate hierarchy:

| Group | Default decimals | Rationale |
|-------|-----------------|-----------|
| Base probabilities (`qx`, `px`, `lx`, `dx`, `tpx`, `tqx`, `ix`, `ox`) | **15** | Foundation of all calculations; maximum float64 precision preserved |
| Commutation functions (`Dx`, `Nx`, `Sx`, `Cx`, `Mx`, `Rx`, `Lx`, `Tx`) | **10** | Large-magnitude accumulated sums; 10 decimals is sufficient |
| Final outputs (`ex`, `annuities`, `insurances`) | **15** | Users can round as needed without losing information |

The 15-decimal default ensures that extreme ages ($l_{111} \approx 4.5 \times 10^{-5}$) and
continuous integration paths do not silently round intermediate values to zero.

## Viewing current settings

```python
from lactuca import LifeTable

lt = LifeTable("PASEM2020_Rel_1o", "m")

# Access via properties on the table
print(lt.decimals.qx)         # 15
print(lt.decimals.annuities)  # 15
print(lt.decimals.Dx)         # 10
```

Alternatively, query the singleton directly without a table instance:

```python
from lactuca import config

print(config.decimals.qx)         # 15
print(config.decimals.annuities)  # 15
print(config.decimals.Dx)         # 10
```

(write-semantics)=
## Changing settings

Decimal properties on table objects (`lt.decimals.*`) are **read-only**.  Attempting to
assign a value raises `AttributeError`:

```python
from lactuca import LifeTable

lt = LifeTable("PASEM2020_Rel_1o", "m")

lt.decimals.annuities = 8
# AttributeError: decimals.annuities is read-only on a table instance.
# To change globally: config.decimals.annuities = 8
```

To change precision, write to the `Config` singleton.  Both import styles are
equivalent — `Config()` and `config` are the same object:

```python
from lactuca import Config, config

# Using Config() — the constructor always returns the same singleton
Config().decimals.annuities = 8

# Using the pre-created alias — identical effect
config.decimals.qx = 12

# Flat-key form (useful in scripts and batch changes)
config.set("decimals_annuities", 8)
config.set_many({"decimals_annuities": 8, "decimals_qx": 12})

# Verify
print(config.decimals.annuities)   # 8
```

All changes take effect immediately and globally for all subsequent calculations in the
process.

## TOML configuration file

Settings can also be loaded from a TOML file — useful for notebooks and shared project
defaults.  Decimal keys go under `[decimals]` **without** the `decimals_` prefix:

```toml
# lactuca_config.toml
[decimals]
annuities  = 6
insurances = 6
qx         = 10

[calculation]
calculation_mode = "continuous_precision"
```

```python
from lactuca import config

config.load("lactuca_config.toml")
```

:::{note}
The `config_path` argument to `Config(config_path=...)` is only respected on the
**first** instantiation of the singleton.  Once the singleton exists, the argument is
silently ignored.  Use `config.load(...)` to reload settings reliably at any point in
the session.
:::

See {doc}`configuration` for the complete settings reference and TOML format.

## Full list of decimal settings

| Config key | `lt.decimals.X` | Affects |
|-----------|-----------------|---------|
| `decimals_lx` | `lx` | `lx(x)` |
| `decimals_dx` | `dx` | `dx(x)` |
| `decimals_qx` | `qx` | `qx(x)`, `tqx(x,t)` |
| `decimals_px` | `px` | `px(x)`, `tpx(x,t)` |
| `decimals_tpx` | `tpx` | `tpx(x,t)` |
| `decimals_tqx` | `tqx` | `tqx(x,t)` |
| `decimals_ix` | `ix` | `ix(x)` on `DisabilityTable` |
| `decimals_ox` | `ox` | `ox(x)` on `ExitTable` |
| `decimals_Lx` | `Lx` | `Lx(x)` |
| `decimals_Tx` | `Tx` | `Tx(x)` |
| `decimals_ex` | `ex` | `ex(x)`, `ex_continuous(x)` |
| `decimals_Dx` | `Dx` | `Dx(x, ir)` |
| `decimals_Nx` | `Nx` | `Nx(x, ir)` |
| `decimals_Sx` | `Sx` | `Sx(x, ir)` |
| `decimals_Cx` | `Cx` | `Cx(x, ir)` |
| `decimals_Mx` | `Mx` | `Mx(x, ir)` |
| `decimals_Rx` | `Rx` | `Rx(x, ir)` |
| `decimals_annuities` | `annuities` | `ax`, `äx`, all annuity methods |
| `decimals_insurances` | `insurances` | `Ax`, `nEx`, all insurance/endowment methods |

:::{note}
`lt.decimals.X` is **read-only** on all table instances — see {ref}`write-semantics` above.
`ix` and `ox` are not available on `LifeTable`; the full availability matrix is in
{ref}`available-properties-by-table-type`.
:::

(available-properties-by-table-type)=
## Available properties by table type

Each table class exposes only the decimal properties that are relevant to its calculation
set.  Accessing a blocked property raises `AttributeError` with a descriptive message.

| Property | `LifeTable` | `DisabilityTable` | `ExitTable` |
|----------|:-----------:|:-----------------:|:-----------:|
| `lx`, `dx`, `px`, `tpx`, `tqx` | ✅ | ✅ | ✅ |
| `Lx`, `Tx` | ✅ | ❌ | ❌ |
| `qx` | ✅ | ❌ | ❌ |
| `ix` | ❌ | ✅ | ❌ |
| `ox` | ❌ | ❌ | ✅ |
| `ex` | ✅ | ❌ | ❌ |
| `annuities`, `insurances` | ✅ | ❌ | ❌ |
| `Dx`, `Nx`, `Sx`, `Cx`, `Mx`, `Rx` | ✅ | ❌ | ❌ |

Example — attempting to read a blocked property:

```python
from lactuca import LifeTable

lt = LifeTable("PASEM2020_Rel_1o", "m")
print(lt.decimals.ix)   # AttributeError: decimals.ix is not available for LifeTable.
```

## Rounding implementation

All rounding uses `np.round(value, decimals)`.  Lactuca applies rounding at exactly two
points in each calculation chain:

1. **Table build time** — the `lx` column is rounded to `decimals.lx` decimal places when
   the table is first constructed.  In both continuous calculation modes
   (`"continuous_precision"` and `"continuous_simplified"`), the full-precision survivor
   values are kept in a separate internal store and used instead, so lx rounding does not
   affect continuous annuity and insurance calculations.
2. **Public method output** — each public method rounds its return value once, using the
   setting that matches the quantity returned.

No additional intermediate rounding occurs between these two points.  For example,
`tpx(x, t)` computes ${}_{t}p_x = l_{x+t}/l_x$ from the rounded `lx` column and
rounds only the final result to `decimals.tpx` decimal places.

## See also

- {doc}`configuration` — complete settings reference and TOML file format
- {doc}`numerical_precision` — why internal calcs use float64 end-to-end
- {doc}`qx_derivation_flow` — where rounding boundaries occur in the derivation chain

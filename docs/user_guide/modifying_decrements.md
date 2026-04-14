# Modifying Decrements

Lactuca lets you apply actuarial adjustments to any decrement table without reloading the
underlying `.ltk` file. The adjustment system is uniform across all table types: one method call,
one dict, five possible keys.

## Method names per table type

Each table class exposes a method named after its own rate symbol:

| Class | Method | Rates modified |
|---|---|---|
| `LifeTable` | `modify_qx(modifications)` | Mortality rates $q_x$ |
| `DisabilityTable` | `modify_ix(modifications)` | Disability incidence rates $i_x$ |
| `ExitTable` | `modify_ox(modifications)` | Exit/turnover rates $o_x$ |

Calling `modify_qx` on a `DisabilityTable` or `ExitTable` raises `NotImplementedError`.
Throughout this guide `LifeTable` / `modify_qx` are used for brevity; all five keys work
identically for `modify_ix` and `modify_ox`.

## Core semantics

**Each call replaces, not accumulates.** Every `modify_*` call starts from the original base
rates loaded from the `.ltk` file, regardless of any previous call. To build a composite
modification, all steps must appear in **one dict**:

```python
from lactuca import LifeTable

lt = LifeTable("PASEM2020_Rel_1o", "m")

# CORRECT: all keys applied in sequence within a single call
lt.modify_qx({
    "age_shift": 2,
    "decrement_multiplier": 1.05,
    "aggravated_risk": 1.2,
})

# WRONG: second call silently overwrites the first
lt.modify_qx({"age_shift": 2})
lt.modify_qx({"decrement_multiplier": 1.05})  # starts from base; age_shift is gone
```

**Keys are applied in insertion order** (Python dict order, guaranteed since Python 3.7).
Different orderings may produce different results; choose the order that matches
the intended actuarial transformation.

**Modifications propagate automatically.** After `modify_*`, every derived quantity
— `lx`, `tpx`, `tqx`, annuity values (`äx`, `ax`), insurance values (`Ax`),
commutation functions (`Dx`, `Nx`, …), life expectancy (`ex`, `ex_continuous`)
— reflects the new rates. No manual refresh or re-instantiation is needed.

**Atomic on failure.** If a `modify_*` call raises (invalid key, wrong type,
mismatched sex, etc.) the table is left in its pre-call state — either still
unmodified or still with the previously applied modification.

**Unknown keys raise immediately.** An unrecognised dict key raises `ValueError` on the
first iteration — it does not silently pass through.

## State and lifecycle

**The primary inspection tool is `summary()`**, which always shows the current state:

```python
from lactuca import LifeTable

lt = LifeTable("PASEM2020_Rel_1o", "m")
lt.modify_qx({"age_shift": 2, "decrement_multiplier": 1.05})
print(lt.summary())
```

When a modification is active, `summary()` adds two sections:

- `Modified: True` and `Modifications applied: age_shift=2, decrement_multiplier=1.05`
- Sample qx values showing both the modified value and the original in parentheses:
  `qx(0) = 0.00012642 (original: 0.0020038)`

Without a modification, sample values show the base rates without parenthetical comparison.

For programmatic inspection, use the read-only properties:

```python
from lactuca import LifeTable

lt = LifeTable("PASEM2020_Rel_1o", "m")
lt.modify_qx({"age_shift": 2, "decrement_multiplier": 1.05})
print(lt.modified)                # True
print(lt.modifications_applied)   # ['age_shift=2', 'decrement_multiplier=1.05']
```

`modifications_applied` is reset at the start of each `modify_*` call and on
`reset_modifications()`.

`lt.w` — **effective** omega after the last modification (decreases after `age_shift`);
`lt.omega` — **original** omega from the file (immutable).

**Resetting** restores all base rates and clears caches:

```python
from lactuca import LifeTable

lt = LifeTable("PASEM2020_Rel_1o", "m")
lt.modify_qx({"decrement_multiplier": 1.05})
lt.reset_modifications()
print(lt.qx(65))  # back to base rate
```

**Reassigning table properties resets modifications.** Assigning any of `sex`,
`cohort`, `duration`, or `unisex_blend` to a new value rebuilds the base decrement
from scratch and sets `modified = False`. Any modification applied before the
reassignment is lost. To re-apply it, call `modify_*` again after the assignment.

**Snapshotting with `copy()`.** To experiment with several modification scenarios
without reinstantiating, take a deep copy before modifying:

```python
from lactuca import LifeTable

lt_base = LifeTable("PASEM2020_Rel_1o", "m")
lt_v1 = lt_base.copy()
lt_v2 = lt_base.copy()

lt_v1.modify_qx({"decrement_multiplier": 1.05})
lt_v2.modify_qx({"aggravated_risk": 1.3})
# lt_base is untouched

print(lt_v1.qx(50), lt_v2.qx(50), lt_base.qx(50))
```

---

## Available modification keys

### `age_shift`

Shift the age axis forward by $n$ integer years. The first $n$ entries are dropped:

$$q_x^{\text{shifted}} = q_{x+n}, \quad x \geq 0$$

The effective upper age `lt.w` decreases by $n$; `lt.omega` is never changed.

```python
from lactuca import LifeTable

lt = LifeTable("PASEM2020_Rel_1o", "m")
original_omega = lt.w
print(f"Original omega: {original_omega}")

lt.modify_qx({"age_shift": 5})
# A person aged 60 now uses rates originally at age 65.
print(lt.w)  # omega_original - 5
```

`age_shift` must be a Python `int` in $[0, \omega]$. Useful for select-and-ultimate
approximations, minimum entry-age products, or simulating older-entry populations.

---

### `decrement_multiplier`

Multiply all rates by a scalar or an age-specific array:

$$q_x^{\text{mod}} = \alpha \cdot q_x$$

```python
import numpy as np
from lactuca import LifeTable

lt = LifeTable("PASEM2020_Rel_1o", "m")

# Uniform 5 % worsening
lt.modify_qx({"decrement_multiplier": 1.05})

# Age-specific: +10 % for ages 50–69, neutral elsewhere
factors = np.ones(lt.w + 1)
factors[50:70] = 1.10
lt.modify_qx({"decrement_multiplier": factors})
```

- **Scalar**: any positive `int`, `float`, or NumPy scalar.
- **Array**: list, tuple, or `ndarray` of positive finite values, length equal to the
  current rate array (`lt.w + 1`, which decreases after an `age_shift`).
- Values are clipped to 1 at the final safety step; intermediate maxima above $10^6$
  are rejected as likely input errors.

---

### `decrement_geometric_increase`

Apply a geometric multiplier to all ages **after** a given pivot age:

$$q_x^{\text{mod}} = q_x \cdot (1 + c)^{x - x_0}, \quad x > x_0$$

Argument: tuple `(c, x_0)`.

- `c` — per-step proportion, any numeric scalar (`int`, `float`, or NumPy numeric) in $[-1.0,\, 1.0]$.
  Use negative values for a geometric *decrease* (e.g. `-0.02` for 2 % annual reduction).
- `x_0` — pivot age, **must be a Python `int`** in $[0, \omega)$. Rates at ages $\leq x_0$
  are unchanged.

```python
from lactuca import LifeTable

lt = LifeTable("PASEM2020_Rel_1o", "m")

# Geometric increase: +2 % per year from age 70 onwards
lt.modify_qx({"decrement_geometric_increase": (0.02, 70)})

# Geometric decrease: -1 % per year from age 40 onwards
lt.modify_qx({"decrement_geometric_increase": (-0.01, 40)})
```

The approximate cumulative growth $(1+c)^{\omega - x_0}$ must be finite and $\leq 10^{12}$.
Rates pushed above 1 are clipped age by age.

---

### `aggravated_risk`

Apply the substandard-life transform to survival probabilities:

$$p_x^{\text{mod}} = p_x^{\,\alpha} \implies q_x^{\text{mod}} = 1 - (1 - q_x)^{\,\alpha}$$

```python
from lactuca import LifeTable

lt = LifeTable("PASEM2020_Rel_1o", "m")
lt.modify_qx({"aggravated_risk": 1.5})   # moderately impaired life
lt.modify_qx({"aggravated_risk": 1.2})   # lightly impaired
```

`aggravated_risk` accepts any positive numeric scalar (`int`, `float`, or NumPy numeric), up to
$100$. Values above 100 are rejected as likely input errors.

---

### `table_combination`

Combine the table with one or more other decrement tables using the
**independent competitive risks** formula:

$$q^{\text{combined}} = 1 - \prod_{i}\,(1 - q_i)$$

where the product runs over the base table and every table in the argument.
This is the exact result for independent decrements and avoids any intermediate rounding.

**Argument forms** — all three are equivalent for a single extra table:

```python
# Single DecrementTable
lt.modify_qx({"table_combination": et})

# List (use this when combining with more than one table)
lt.modify_qx({"table_combination": [et, dt]})

# Tuple
lt.modify_qx({"table_combination": (et, dt)})
```

#### Allowed type combinations

The combination matrix is actuarially motivated:

| Base table | Can combine with |
|---|---|
| `LifeTable` | `ExitTable`, `DisabilityTable` |
| `DisabilityTable` | `ExitTable` |
| `ExitTable` | `ExitTable` |

Combinations not in this matrix raise `ValueError`.

:::{note}
The matrix reflects standard actuarial practice: a life table is the dominant
decrement; it makes sense to *enrich* it with exit or disability causes, but combining
two life tables (same cause) has no actuarial meaning. Similarly, an ExitTable is a
secondary decrement that can only be combined with another ExitTable (two independent
exit causes) or used as the *other* table from a LifeTable or DisabilityTable perspective.
:::

**The matrix is directional.** It is the `table_type` of the *base* table (the one
you call `modify_*` on) that determines what is allowed, not the other way around.
For example, `lt.modify_qx({"table_combination": et})` is valid (Life absorbs Exit),
but `et.modify_ox({"table_combination": lt})` raises `ValueError` (Exit cannot absorb Life).

#### Additional constraints

All tables in the combination must share the same **sex**. Beyond that, two optional checks
apply when both tables carry the relevant metadata:

- **Cohort mismatch**: if both tables have a non-`None` cohort and they differ,
  combining is rejected (`ValueError`). A period table (`cohort=None`) can always
  combine with a generational table.
- **Duration mismatch**: if both tables have a numeric duration and they differ,
  combining is rejected. `duration=None` or `"ult"` can combine freely.

#### Single-table combination example

```python
from lactuca import LifeTable, ExitTable

lt = LifeTable("PASEM2020_Rel_1o", "m")
et = ExitTable("DummyEXIT", "m")

# Combine: life + exit (independent risks)
lt.modify_qx({"table_combination": et})
# lt.qx(x) now returns the probability of decrement from EITHER cause
# (death OR exit) — the net single decrement in a multiple-decrement framework.
```

:::{note}
`DummyEXIT` and `DummySD2015` are bundled test tables provided for experimentation
and unit testing. In production you would load an `ExitTable` and `DisabilityTable`
from your own `.ltk` files using `TableBuilder`.
:::

#### Combining three independent decrements

A common case in pension actuarial work is the active-member table, which combines
three independent decrements:
mortality, disability onset, and voluntary exit. Pass them in one call:

```python
from lactuca import LifeTable, DisabilityTable, ExitTable

lt = LifeTable("PASEM2020_Rel_1o", "m")
dt = DisabilityTable("DummySD2015", "m")
et = ExitTable("DummyEXIT", "m")

# Single call — q = 1 − (1−q_life)(1−q_disability)(1−q_exit)
lt.modify_qx({"table_combination": [dt, et]})
```

If the tables have different lengths, all are first truncated to the shortest one.
The combined array is then truncated further at the first age where $q_x = 1$.

---

## Combining modifications

Any subset of the five keys can appear in a single dict. They are applied in insertion
order; order is significant:

```python
# Age-shifted, loaded, and combined — all in one call
from lactuca import LifeTable, ExitTable

lt = LifeTable("PASEM2020_Rel_1o", "m")
et = ExitTable("DummyEXIT", "m")

lt.modify_qx({
    "age_shift": 2,
    "decrement_multiplier": 1.05,
    "table_combination": et,
})
```

Example showing order dependence:

```python
from lactuca import LifeTable

lt = LifeTable("PASEM2020_Rel_1o", "m")

# Order A: scale first, then aggravated risk
# result = 1 - (1 - 1.05 * qx)^1.5
lt.modify_qx({"decrement_multiplier": 1.05, "aggravated_risk": 1.5})
print(lt.qx(60))  # result A

# Order B: aggravated risk first, then scale
# result = 1.05 * (1 - (1 - qx)^1.5)
lt.modify_qx({"aggravated_risk": 1.5, "decrement_multiplier": 1.05})
print(lt.qx(60))  # result B — different from A
```

---

## Precision

All intermediate operations run in float64. The final array is:

1. Checked for non-finite values — any NaN or Inf raises `ValueError`.
2. Clipped to $[0, 1]$.
3. Rounded to the table-type decimal precision:
   - `LifeTable` → `config.decimals.qx`
   - `DisabilityTable` → `config.decimals.ix`
   - `ExitTable` → `config.decimals.ox`

See {doc}`decimals_rounding` for how to configure these precision settings.

---

## See also

- {doc}`using_tables` — how to instantiate and configure tables
- {doc}`numerical_precision` — float64 and rounding policy
- {doc}`decimals_rounding` — how to configure decimal precision


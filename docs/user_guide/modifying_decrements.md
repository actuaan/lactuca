# Modifying Decrements

Lactuca lets you apply actuarial adjustments to any decrement table without reloading the
underlying `.ltk` file. The adjustment system is uniform across all table types: one method call,
one dict, six possible keys.


### `table_combination` — competing risks

Combines decrement arrays with the **product formula** $q = 1 - \prod_i (1 - q_i)$
for **independent competing risks** (default). Optional `combination_mode="udd"`
declares the UDD multiple-decrement assumption; v1 stores the same collapsed total
$q_x^{\mathrm{comb}}$ for two or three causes (associated singles $q'^{(j)}$ are not
returned). Pairwise sums $q_1 + q_2$ may exceed 1 while the combined probability
remains valid (e.g. $0.6$ and $0.6$ → $0.84$). Validation rejects only the
**combined** probability above 1 (numerical tolerance).

Array index $i$ is **integer age** $x = i$ (see {ref}`table-combination-age-alignment` below).
The same combined $q_x$ feeds all four calculation modes (`discrete_precision`,
`discrete_simplified`, `continuous_precision`, `continuous_simplified`). Modes differ in
how present values are computed, not in the underlying decrement table. All four modes remain
**actuarially coherent** (same product and conventions) but are **not** required to yield
numerically identical results — see {ref}`actuarial-coherence-of-modes` in
{doc}`calculation_modes`.

Optional sibling key **`combination_mode`** (`"independent"` default, or `"udd"`) declares the actuarial assumption when combining tables; see {ref}`combination-mode` below.

## Method names per table type

Each table class exposes a method named after its own rate symbol:

| Class | Method | Rates modified |
|---|---|---|
| `LifeTable` | `modify_qx(modifications)` | Mortality rates $q_x$ |
| `DisabilityTable` | `modify_ix(modifications)` | Disability incidence rates $i_x$ |
| `ExitTable` | `modify_ox(modifications)` | Exit/turnover rates $o_x$ |

Calling `modify_qx` on a `DisabilityTable` or `ExitTable` raises `NotImplementedError`.
Throughout this guide `LifeTable` / `modify_qx` are used for brevity; all six keys work
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

**Public API after `age_shift`.** The stored array is shortened: index $i$ holds
rates for **calendar age** $n + i$ when `start_age = 0` (not calendar age $i$).
Example: after `{"age_shift": 40}`, `lt.qx(0)` returns $q_{40}$ and `lt.qx(10)`
returns $q_{50}$. When combining in the same dict, alignment uses the same offset
(see {ref}`table-combination-age-alignment`).

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

(combination-mode)=
#### `combination_mode`

Optional key in the same dict as `table_combination`. It declares the **actuarial
assumption** when merging decrement tables; it does not change which tables may be
combined.

| Value | Meaning |
|---|---|
| *(omitted)* | Same as `"independent"`; not recorded in `modifications_applied` |
| `"independent"` | Independent competing risks (product formula below) |
| `"udd"` | Uniform Distribution of Decrements (UDD) — Lactuca applies the correct
  associated-single formula for two or three causes internally |

**Default.** Omitting the key is equivalent to `"independent"` and matches pre-v1
behaviour. Pass `"independent"` explicitly when you need traceability in
`summary()` / `modifications_applied`.

**Mode `"udd"`.** Host + **one** other table (two causes) uses the two-way UDD
associated singles; host + **two** others (three causes, e.g. *Masa Activa* with
`[dt, et]`) uses the three-way Bowers/Jordan formula. Host + **three or more**
others raises `ValueError` in v1.

**v1 collapsed total.** For two or three causes, `"independent"` and `"udd"`
produce the **same** stored vector $q_x^{\mathrm{comb}}$ (algebraic identity). The
difference is **declarative** — which assumption is recorded. Cause-specific
associated rates $q'^{(j)}$ are **not** returned in v1 (planned for a future release).

**Key order.** `combination_mode` may appear before or after `table_combination` in
the dict (same result). Other keys (`decrement_multiplier`, `age_shift`, …) still
depend on insertion order — see {ref}`combining-modifications-order` below.

**Order of `others`.** For the collapsed total $q_x^{\mathrm{comb}}$ in v1, permuting
the list of other tables (e.g. `[dt, et]` vs `[et, dt]`) yields the **same** result
up to float64 rounding noise — the product and UDD collapsed formulas are commutative
in the other causes when the host is fixed. List order **does not** change stored
$q_x^{\mathrm{comb}}$; dict key order for other modifications still matters (see above).

**UDD formulas (v1 collapsed total).** For two causes with host $q^{(0)}$ and one other
$q^{(1)}$:

$$q'^{(1)} = q^{(0)}\left(1 - \frac{q^{(1)}}{2}\right), \quad
q'^{(2)} = q^{(1)}\left(1 - \frac{q^{(0)}}{2}\right), \quad
q^{\mathrm{comb}} = q'^{(1)} + q'^{(2)}$$

For three causes (host plus two others), Lactuca sums three associated
singles (Bowers/Jordan); the total equals the independent product above.

:::{note}
Under UDD, the summed associated single decrements equal the independent product
formula for two or three causes. Lactuca selects the UDD formula automatically
when `combination_mode="udd"`.
:::


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
- **Generational `base_year`**: when host and other are both generational with
  different `base_year`, Lactuca emits a `UserWarning` (each table keeps its own
  projection before merge; no cross-table base-year validation).
- **Select-ultimate host**: `table_combination` merges the host’s
  **duration-specific base column** fixed at construction time (the same rates
  loaded for the `duration` argument passed to `LifeTable(...)`). It does not
  re-resolve select columns after `modify_*`. Combine before relying on a
  different duration column, or build a separate table instance per duration.
- **Duplicate instances**: the host cannot appear in `others`, and the same
  table instance cannot be listed twice (`[et, et]` raises `ValueError`). Two
  **distinct** instances with identical rates remain valid (e.g. sensitivity with
  two independent exit causes).

(table-combination-age-alignment)=
#### Age alignment and table length

Lactuca aligns tables by **calendar age**, not by “shortest array wins”.

**Index convention.** For valid `.ltk` tables, decrement array index $i$ corresponds to
integer age $x = i$. Tables with `start_age > 0` (e.g. disability tables starting at
age 20) store **zero padding** in indices $0,\ldots,\text{start\_age}-1$; index 25 is
always age 25 for every table type.

**Host table.** The table you call `modify_*` on is the **host**. Its decrement array
length after any prior keys in the same dict (e.g. `age_shift`) is preserved through
the combination step, except for the terminal rule below.

**`age_shift` before `table_combination`.** When both keys appear in one dict, a prior
`age_shift` of $n$ years is tracked as a calendar-age offset: host row index $i$
combines with *other* rates at calendar age $x = i + n$, not at index $i$. This
matches the shifted semantics $q_x^{\text{shifted}} = q_{x+n}$ and makes
`{"age_shift": n, "table_combination": …}` actuarially equivalent to combining
first and then shifting.

**Shorter *other* tables.** If an *other* table has a lower terminal age $\omega$ than
the host, ages above that $\omega$ contribute **implicit** $q_{\text{other}} = 0$
(no additional competing risk). Example: Life ($\omega = 108$) + Exit ($\omega = 100$)
uses only mortality for ages 101–108 **until** the combined rate reaches 1.

**Per-age formula** (host index $i$, calendar age $x = i + n$ when a prior
`age_shift` of $n$ was applied in the same dict; otherwise $n = 0$):

$$q^{\text{comb}}_x = 1 - (1 - q^{\text{host, shifted}}_i) \prod_{j} (1 - q^{(j)}_x)$$

where each $q^{(j)}_x$ is taken from the *other* table at calendar age $x$ when $x$
is within that table’s array; otherwise the factor $(1 - q^{(j)}_x)$ is $1$.

**Truncation at certainty.** After combination, the array is truncated at the **first**
index where $q^{\text{comb}} \approx 1$ (tolerance $10^{-12}$; includes $\omega$ rows
where the *other* table has $q = 1$, e.g. mandatory exit at retirement age). This may
shorten the host below its original $\omega$ even when implicit zero applied above.
When a prior `age_shift` of $n$ was applied in the same dict, the `UserWarning` and
`summary()` note report both the **truncation index** in the shortened array and the
**calendar age** $n + \text{index}$ (e.g. index 60, calendar age 100 after
`age_shift=40` with exit at age 100).

(table-combination-truncation)=
### Combined $q = 1$ truncation

:::{important}
**Two length rules (do not confuse them).**

1. **Shorter *other* $\omega$** — ages above the other table’s terminal age use
   implicit $q_{\text{other}} = 0$; the host keeps its length until rule 2 applies.
2. **Combined $q = 1$** — the result is truncated at that age; `w` may drop below the
   host’s pre-combination $\omega$.

**Worked example (bundled tables).** `PASEM2010` (male) has $\omega = 112$; `DummyEXIT`
has mandatory exit $o_x = 1$ at age 100 ($\omega = 100$). After
`lt.modify_qx({"table_combination": et})`:

- `lt.w` becomes **100** (not 112).
- Ages 101–112 are removed because $q^{\text{comb}}_{100} = 1$.
- Lactuca emits a `UserWarning` and records the truncation in `summary()`.

```python
from lactuca import LifeTable, ExitTable

lt = LifeTable("PASEM2010", "m")
et = ExitTable("DummyEXIT", "m")
w_before = lt.w  # 112

lt.modify_qx({"table_combination": et})

assert lt.w == et.w == 100
assert w_before > lt.w
print(lt.summary())  # includes Note: table_combination truncated omega (112->100 ...)
```

Use `warnings.filterwarnings` in batch pipelines if you intentionally combine with a
table that forces exit before the host’s natural $\omega$.
:::

**Rejected inputs** (`ValueError`):

| Condition | Reason |
|---|---|
| Incompatible `table_type` pair | See matrix below |
| Different `sex` | |
| Both cohorts set and differ | Period (`cohort=None`) combines freely |
| Both numeric `duration` and differ | `"ult"` / `None` combines freely |
| `len(other._decrement) ≠ len(other._decrement_base)` | *Other* table shortened (often by `age_shift` on that table) — apply `age_shift` on the table you are modifying instead |
| Positive rates in *other* padding below `start_age` | Cannot align by age index |
| Positive rates in *host* padding below `start_age` | Cannot align by age index (includes partial `age_shift`: calendar ages `[n, start_age)` must be zero when `age_shift=n`) |
| Host or other rate outside $[0, 1]$ before combine | Invalid input — raised instead of silent clip |
| Combined $q > 1$ at any aligned age | Invalid competing risks (message reports index and calendar age when `age_shift` preceded combine) |
| Host instance in `others` or duplicate instance in `others` | Same object twice inflates decrements — use distinct instances |
| `combination_mode` without `table_combination` | Orphan key |
| `combination_mode="udd"` with 4+ causes | UDD limited to 2–3 causes in v1 |
| Unknown `combination_mode` (`"udd_2"`, `"UDD"`, bool, etc.) | Invalid literal |

**`age_shift` on the *other* table.** Do not call `age_shift` on a table you only
pass into `table_combination`. That shortens the *other* array and raises
`ValueError`. To evaluate combined rates from age $x$ onward on the table you are
modifying, use `{"age_shift": x, "table_combination": other}` on **that** table
(e.g. `LifeTable.modify_qx`), not `other.modify_*` first.

**Beyond-$\omega$ API vs combination.** For an *other* table shorter than the
table you are modifying, the public accessor `other.ix(age)` / `other.ox(age)` may
return `1.0` when `age` exceeds that table’s $\omega$ (standard beyond-$\omega$
convention). **Combination does not use that value:** it reads `other._decrement`
and treats ages above the *other* $\omega$ as **implicit** $q_{\text{other}} = 0$.
When validating by hand, use aligned base-array values, not beyond-$\omega$ API
returns. Example: `DummySD2015` has $\omega = 65$; at age 75, `dt.ix(75)` is `1.0`
but combination uses $i_{75} = 0$, so `lt.qx(75)` equals life mortality only.

**Partial MDDT support (v1).** `table_combination` with
`combination_mode="udd"` declares the UDD multiple-decrement assumption and stores
the **collapsed total** decrement $q_x^{\mathrm{comb}}$ for integer ages.
Cause-specific associated rates $q'^{(j)}$ are not returned in the public API in v1.

**Fractional-year survival.** After combination (both `independent` and `udd` in v1),
fractional `lx`, `px(m>1)`, and `tpx` use `config.lx_interpolation` on the collapsed
total $q_x^{\mathrm{comb}}$ stored in the host table. With default linear (UDD)
interpolation this is ${}_s p_x = 1 - s\,q_x^{\mathrm{comb}}$ within each year,
coherent with the annual recursion $l_{x+1} = l_x(1-q_x^{\mathrm{comb}})$.
The `udd` mode records the UDD associated-single **assumption** used to build
$q^{\mathrm{comb}}$ at combine time; fractional survival matches `independent`
while v1 stores only the collapsed vector.

**Masa Activa (aggregated, not Markov).** Combining life + disability + exit treats
mortality, disability incidence ($i_x$), and exit ($o_x$) as **mutually exclusive
decrements in the same policy year** (competing risks on the active population).
This is standard for simplified active-member tables; it is **not** a multi-state
Markov model (no disabled-state recovery, no separate disabled-life mortality path).

**Data source.** Combination reads each `other._decrement` (the **active** float64
vector on that table instance), not `other._decrement_base`. If the other table was
previously modified (e.g. `et.modify_ox({"decrement_multiplier": 1.1})`), those
adjusted rates are combined. For file/base rates, call `other.reset_modifications()`
or pass a fresh instance. Combination does **not** use the public `qx()` / `ix()` /
`ox()` accessors (rounded; beyond-$\omega$ returns may differ from alignment).

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

# Same total q in v1, UDD assumption recorded explicitly (Masa Activa)
lt.modify_qx({"table_combination": [dt, et], "combination_mode": "udd"})
```

Life + Disability: incidence $i_x$ at age 25 combines with mortality $q_x$ at the
same index (age 25). Ages above the disability table’s $\omega$ use only host
mortality (implicit $i_x = 0$). Ages where the exit table has $o_x = 1$ yield
combined $q_x = 1$ and truncate the result there.

#### `combination_mode` examples

```python
from lactuca import LifeTable, DisabilityTable, ExitTable
import pytest

lt = LifeTable("PASEM2020_Rel_1o", "m")
et = ExitTable("DummyEXIT", "m")
dt = DisabilityTable("DummySD2015", "m")

# Default — independent; combination_mode not listed in modifications_applied
lt.modify_qx({"table_combination": et})

# UDD two causes
lt.modify_qx({"table_combination": et, "combination_mode": "udd"})

# Explicit independent — appears in summary() / modifications_applied
lt.modify_qx({"table_combination": et, "combination_mode": "independent"})

# Disability host + exit (two causes)
dt.modify_ix({"table_combination": et, "combination_mode": "udd"})

# Errors (ValueError)
lt.modify_qx({"combination_mode": "udd"})  # orphan key
lt.modify_qx({"table_combination": [dt, et, et], "combination_mode": "udd"})  # 4+ causes
```

See {doc}`../errors_reference` for exact message patterns.


---

(combining-modifications-order)=
## Combining modifications

Any subset of the six keys can appear in a single dict. They are applied in insertion
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

**Order matrix (verified behaviour).**

| Key pair | Order significant? | Notes |
|---|---|---|
| `combination_mode` ↔ `table_combination` | No | Pre-scan; same collapsed `q` |
| `age_shift` ↔ `table_combination` | No | Calendar-age alignment (host index $i$ ↔ age $i+n$) |
| `others` list (`[dt, et]` vs `[et, dt]`) | No | Collapsed total $q$ in v1 (float noise only) |
| `decrement_multiplier` ↔ `table_combination` | **Yes** | Scales host **before** vs **after** merge |
| `aggravated_risk` ↔ `decrement_multiplier` | **Yes** | Non-linear survival transform vs linear scale |
| `decrement_geometric_increase` ↔ `age_shift` | **Yes** | Tail indices depend on pre-shift length |
| `decrement_geometric_increase` ↔ `table_combination` | **Yes** | Geometric tail applies to pre-merge vs post-merge rates (different $q_x$) |
| `decrement_multiplier` (vector) after `table_combination` | Length must match | Array length follows the table **after** prior keys (may differ from base $\omega$) |
| `table_combination` vs `decrement_multiplier` / `decrement_geometric_increase` | **Yes** (clip policy) | `table_combination` **raises** if combined $q > 1$ or if host/other rates are outside $[0, 1]$ before merge; multiplier/geometric rates above 1 are **clipped** at the final safety step |

---

## Precision

All intermediate operations run in float64. The final array is:

1. Checked for non-finite values — any NaN or Inf raises `ValueError`.
2. Clipped to $[0, 1]$ (including rates pushed above 1 by `decrement_multiplier` or
   `decrement_geometric_increase`; `table_combination` raises instead of clipping when
   host/other rates are outside $[0, 1]$ or combined $q > 1$ before this step).
3. Truncated at the **first** index where $q \approx 1$ **and** a positive tail
   survives ($0 < q_{x+1} < 1$), keeping `lx` / `ex` coherent with
   $l_{x+1} = l_x(1-q_x)$. Terminal plates ($q_\omega \approx 1$ only) are
   unchanged. A `UserWarning` is emitted when $\omega$ drops; see
   {ref}`table-combination-truncation`.
4. Rounded to the table-type decimal precision:
   - `LifeTable` → `config.decimals.qx`
   - `DisabilityTable` → `config.decimals.ix`
   - `ExitTable` → `config.decimals.ox`

See {doc}`decimals_rounding` for how to configure these precision settings.

---

## See also

- {doc}`using_tables` — how to instantiate and configure tables
- {doc}`numerical_precision` — float64 and rounding policy
- {doc}`decimals_rounding` — how to configure decimal precision


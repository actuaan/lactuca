# Numerical Precision

This page explains the numerical precision policy applied throughout Lactuca's calculation engine
and why certain design decisions were made.  It also describes the integer-detection system that
enforces actuarial method contracts (commutation functions require integer ages; continuous
counterparts require fractional ages).

## Precision strategy overview

Maximum actuarial accuracy is achieved through five complementary layers:

| Layer | What it does |
|-------|--------------|
| **Input normalization** | Every numeric input is cast to float64 before entering the pipeline |
| **Dual survivor store** | Continuous calculations use the full-precision survivor function; discrete calculations use the rounded version |
| **Output-only rounding** | No intermediate rounding — the user-configured decimal precision is applied only at public method return |
| **Log-domain joint-life products** | Joint-life survival (two or more lives) is computed in log space to prevent underflow when multiplying per-life probabilities |
| **Suffix-cumsum commutation chains** | Commutation sums ($N_x$, $M_x$, …) are computed in a single O(n) pass using cumulative summation |

See {doc}`decimals_rounding` for how output-only rounding is configured.

## float64 throughout

All internal actuarial values are stored and computed in **IEEE 754 double precision (float64)**,
which provides approximately 15–16 significant decimal digits.

Lactuca never introduces lower-precision intermediate values.  Every numeric input — scalar,
list, or array — is normalized to a 1-D float64 array before it enters the calculation pipeline.
This includes age arguments, time horizons, interest rates, and probability inputs.

## Dual survivor store

Each table maintains two internal stores for the survivor function $l_x$:

| Store | Contents | Used by |
|-------|----------|---------|
| **Rounded survivor values** | Rounded to `decimals.lx` decimal places (default 15) | Public `lx()`, all discrete calculations |
| **Full-precision survivor values** | Unrounded, full float64 precision | All continuous-mode annuity and insurance calculations |

**Why two stores?**

For discrete calculations, the rounded values provide the canonical "official" $l_x$ that matches
published table values and satisfies regulatory rounding requirements.

For continuous calculations (numerical integration over fractional ages), the full-precision
values prevent cumulative rounding error.  With the default 15-decimal rounding, the difference
at any single step is negligible, but it compounds measurably over long annuity integrations
reaching up to $\omega - x$ steps.

Both `"continuous_precision"` and `"continuous_simplified"` use the full-precision store for
annuity and insurance calculations.  The sole exception is the **average-force endowment
approximation** in `"continuous_simplified"` mode.  This method evaluates the pure endowment as

$${}_{n}E_x = \exp\!\left[-n\!\left(\bar{\delta} + \bar{\mu}_{x:\overline{n}|}\right)\right]$$

where the combined force is constructed from two average-force components:

- $\bar{\delta} = \delta$ — the constant force of interest (exact, from the interest rate).
- $\bar{\mu}_{x:\overline{n}|} = -\dfrac{\ln {}_{n}p_x}{n}$ — the average force of mortality
  over $[x,\, x+n]$, derived from the public (rounded) value of ${}_{n}p_x$.

Because $\bar{\mu}$ is derived from the **rounded** survivor quotient ${}_{n}p_x = l_{x+n}/l_x$,
the rounded survivor values flow through this path rather than the full-precision store
described in the *Dual survivor store* section above.  This is inherent to the approximation
model, not a precision gap: the average-force endowment is defined in terms of a period
survival probability, so using the canonical public ${}_{n}p_x$ is both correct and
numerically stable.

## Log-domain joint-life products

**Single-life survival** is not computed as a year-by-year product.  It uses the closed-form ratio
${}_{t}p_x = l_{x+t} / l_x$ directly from the survivor table — a single division with no
accumulation of per-year factors and therefore no underflow risk.

**Joint-life survival** (two or more lives) requires multiplying the individual probabilities:

$${}_{t}p_{xy} = {}_{t}p_x \cdot {}_{t}p_y$$

For $n$ lives the general form is the product over all $i$:

$${}_{t}p_{x_1 \cdots x_n} = \prod_{i=1}^{n} {}_{t}p_{x_i}$$

Lactuca computes this in log space to prevent floating-point underflow when any
${}_{t}p_{x_i}$ is very small:

$${}_{t}p_{x_1 \cdots x_n} = \exp\!\left(\sum_{i=1}^{n} \ln {}_{t}p_{x_i}\right)$$

For $t = 0$ the argument of each logarithm is $1$ and the sum is $0$, giving
$\exp(0) = 1$ correctly; the formula is applied uniformly across all $t \ge 0$.

## Integer and fractional age detection

Actuarial contracts distinguish between methods that require **integer ages** — commutation
functions ($D_x$, $N_x$, $C_x$, $T_x$, $L_x$, $e_x$) — and those that require **strictly
fractional ages** — their continuous counterparts ($T_x^{\circ}$, $L_x^{\circ}$, $e_x^{\circ}$).

Because actuarial inputs frequently arrive as the result of float64 arithmetic (e.g. an age
accumulated through UDD interpolation), a tolerance is used instead of exact integer comparison:

$$\varepsilon_{\text{int}} = \max\!\left(1 \times 10^{-12},\ \varepsilon_{\text{float64}} \times 10^4\right) \approx 2.22 \times 10^{-12}$$

where $\varepsilon_{\text{float64}} = 2.22 \times 10^{-16}$ is machine epsilon.
Since $\varepsilon_{\text{float64}} \times 10^4 \approx 2.22 \times 10^{-12}$ exceeds the $1 \times 10^{-12}$ floor,
the machine-epsilon term dominates and sets the effective value.  This value:

- **Tolerates** legitimate IEEE 754 rounding from single float64 operations (accumulated error ~$10^{-13}$ to $10^{-12}$, for example from UDD interpolations).
- **Rejects** truly fractional values, which differ from the nearest integer by a margin at least $10^8$ times larger than $\varepsilon_{\text{int}}$.

The rounding function used for integer detection is `np.rint` (ties-to-even), the same function
used consistently whenever a validated integer age is converted for array indexing.

### Tolerance in practice

The following snippet shows when a value is classified as integer-valued and when it is
rejected, using only NumPy — no table required:

```python
import numpy as np

# _EPS_INT ≈ 2.22e-12
eps_int = max(np.finfo(np.float64).eps * 1e4, 1e-12)   # 2.220446049250313e-12

# --- Accepted as integer ---

# 1. Exact integer — distance to nearest integer is zero:
abs(np.float64(50.0) - np.rint(np.float64(50.0)))      # 0.0  ≤  eps_int  →  True

# 2. Float64 arithmetic residual — magnitude ~1e-13, still within tolerance:
age = np.float64(50) + np.float64(1e-13)               # realistic accumulation residual
abs(age - np.rint(age))                                 # ≈ 1e-13  ≤  eps_int  →  True

# --- Rejected as integer ---

# 3. Genuinely fractional age — differs from nearest integer by 0.5:
abs(np.float64(50.5) - np.rint(np.float64(50.5)))      # = 0.5  →  False
# Ratio: 0.5 / eps_int  ≈  2.25 × 10¹¹  (eleven orders of magnitude above the threshold)
```

Case 2 is the key scenario: a float64 value produced by, say, a UDD interpolation loop
may carry a rounding residual of order $10^{-13}$.  That residual is smaller than
$\varepsilon_{\text{int}}$ and therefore transparent to integer detection — the value
behaves as the integer it represents without any manual rounding by the caller.

Case 3 is the contrast: $0.5$ is approximately $2.25 \times 10^{11}$ times larger than
$\varepsilon_{\text{int}}$, so even the tiniest genuinely fractional age is rejected with
a large safety margin.

### Integer contract: commutation functions

All commutation functions validate that input ages are integer-valued before computing.
Passing a fractional age raises `ValueError` with a message that names the method and suggests
the continuous counterpart if one exists:

```python
from lactuca import LifeTable, Config

Config.reset()
lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)

lt.Dx(50.5)
# ValueError: [Dx] Commutation functions require integer ages.
# Got non-integer age(s): [50.5].
```

The same validation applies to all commutation functions.  Correct usage passes an integer age:

```python
from lactuca import LifeTable, Config
import numpy as np

Config.reset()
lt = LifeTable("PASEM2020_Rel_1o", "m", interest_rate=0.03)

# Integer age — accepted:
lt.Dx(50)     # returns D_50 = v^50 · l_50
lt.Nx(50)     # returns N_50 = Σ D_k  for k = 50, …, ω
lt.Mx(50)     # returns M_50 = Σ C_k  for k = 50, …, ω

# Float64 residual from arithmetic — also accepted (within ε_int of 50):
age = np.float64(50) + np.float64(1e-13)
lt.Dx(age)    # identical result to lt.Dx(50, ir=0.03)
```

### Fractional contract: continuous methods

The continuous expectation-of-life and related methods require **strictly fractional** input
ages and raise `ValueError` when an integer is passed, directing the user to the discrete
counterpart:

```python
from lactuca import LifeTable, Config

Config.reset()
lt = LifeTable("PASEM2020_Rel_1o", "m")

lt.ex_continuous(50)   # 50 is integer-valued
# ValueError: [ex_continuous] Requires fractional (non-integer) ages.
# Got integer age(s): [50].
# Use ex() for integer ages instead.
```

Note that passing `50.0` (integer value stored as float) is also rejected — integer detection
is value-based, not type-based:

```python
lt.ex_continuous(50.0)
# ValueError: [ex_continuous] Requires fractional (non-integer) ages.
# Got integer age(s): [50.0].
# Use ex() for integer ages instead.
```

For each integer age there are two complementary calls — one discrete, one continuous:

```python
# Discrete: complete life expectancy at the integer birthday (e.g. exact age 50):
lt.ex(50)              # ė_50  (discrete UDD approximation)

# Continuous: complete life expectancy at a fractional age,
# e.g. six months after the 50th birthday:
lt.ex_continuous(50.5)           # trapezoidal integration, default m=12
lt.ex_continuous(50.5, m=52)     # finer grid: 52 sub-intervals per year (weekly)
```

The `m` parameter controls the number of integration sub-intervals per year of age.
The default `m=12` (monthly grid) is sufficient for all standard actuarial valuations.
Valid values are 1, 2, 3, 4, 6, 12, 52, and 365.

### Hot-path optimization

In the innermost survivor interpolation loop, integer detection is inlined using a
floor-plus-fractional-part pattern rather than calling the centralized detection logic,
because the fractional part is needed immediately for the interpolation itself.
The same $\varepsilon_{\text{int}}$ tolerance is used in both paths, guaranteeing
consistent results throughout the library.

## Why rounding only at output

Internal functions never round intermediate values (the rounded survivor column is the
exception — it is rounded at cache-build time, which is itself an output boundary).
This policy:

1. Prevents error accumulation across chained calculations.
2. Makes intermediate values portable between different decimal precision settings.
3. Ensures that changing `Config.decimals.*` settings affects only the final reported value,
   not the internal computation path.

## Summation for commutation chains

Commutation functions that accumulate many terms — such as

$$N_x = \sum_{k=x}^{\omega} D_k$$

— are computed in a **single O(n) pass** using a reversed cumulative sum (suffix sum): the
$D_k$ array is reversed, a cumulative sum is taken, and then reversed again.  This avoids
any loop overhead and uses NumPy's internally pairwise-accurate accumulation, which bounds
the error at $O(\varepsilon \log_2 n)$ rather than the $O(n \varepsilon)$ of naive sequential
summation.

Rounding to the relevant commutation decimal setting is applied once, after the full chain has
been computed.

Annuity and insurance present values are accumulated using standard NumPy array summation (also
pairwise-accurate internally).  There is no user-configurable summation mode.

## Actuarial invariants

Lactuca enforces actuarial invariants at two points:

**Runtime probability checks** — every time a probability is returned by an internal interval
calculation, it is validated to lie in $[0, 1]$.  A value outside this range raises `ValueError`
rather than silently propagating an out-of-range result.  The check covers $p_x$, $q_x$, and
all ${}_{t}p_x$ values:

- $0 \leq p_x \leq 1$ (equivalently $0 \leq q_x \leq 1$, since $p_x + q_x = 1$ by construction)

A tolerance of $10^{-12}$ is applied to both bounds, so values such as $1.0 + 5 \times 10^{-13}$
produced by UDD or CFM interpolation are silently accepted rather than triggering a false alarm.

**Terminal boundary protection** — at the terminal age $\omega$ the survivor function is $l_\omega = 0$,
which would cause a 0/0 division when computing ${}_{t}p_x = l_{x+t}/l_x$.  The engine uses a
masked division (`np.divide(..., where=lx != 0)`) with a zero-initialised output, so any
survival probability based on an empty cohort evaluates to ${}_{t}p_x = 0$ cleanly, without
producing `NaN`.

**Structural table-build invariants** — the following properties are guaranteed by the table
construction process and are not re-checked on every calculation:

- $l_x$ is non-increasing across all ages (when derived from stored decrement columns)
- For life tables, $q_\omega = 1.0$ at the terminal age (enforced at build/load; rounding
  artefacts within $10^{-4}$ are auto-corrected)

## See also

- {doc}`decimals_rounding` — configuring decimal precision and the output-rounding policy
- {doc}`qx_derivation_flow` — the derivation chain with rounding boundaries
- {doc}`calculation_modes` — how continuous and discrete modes differ in precision

# Last Payment Adjustment

When the product $n \cdot m$ of a term and payment frequency is not an integer, the final
sub-annual period is **incomplete**.  Lactuca applies a proportional weight to that last
period so that the present value reflects the true contracted term $n$ — not the nearest
aligned grid point.

:::{note}
This adjustment applies **only to `discrete_precision` mode** for the standard
``payments_frac`` scaling on the full payment grid.  In ``discrete_simplified``
with fractional ``n_eff`` and ``m > 1``, the terminal fraction ``s = n_eff - k``
is valued on an exact m-thly tail (``discrete_precision`` conventions) rather
than via Woolhouse on the annual grid.  Continuous modes do not implement
fractional final-period scaling on the simplified path.
:::

## Annuity-due versus annuity-immediate

Lactuca exposes separate methods for each payment convention:

| Method family | Convention | Symbol | Fractional adjustment |
|--------------|-----------|--------|----------------------|
| `äx`, `äxy`, `äxyz`, `äjoint` | Annuity-**due** — payments at the **start** of each sub-period | $\ddot{a}^{(m)}$ | **None** — last payment is at $\lfloor nm \rfloor / m \le n$ with full weight |
| `ax`, `axy`, `axyz`, `ajoint` | Annuity-**immediate** — payments at the **end** of each sub-period | $a^{(m)}$ | Last payment scaled by $w$ when $w > 0$ |

For annuity-due methods (`äx`, `äxy`, `äxyz`, `äjoint`), the last payment is at
$\lfloor nm \rfloor / m \le n$ and is included with full weight — no scaling.  The next
payment would fall at $(\lfloor nm \rfloor + 1)/m > n$ and is simply excluded.

For annuity-immediate methods (`ax`, `axy`, `axyz`, `ajoint`), the final period ends at
$(\lfloor nm \rfloor + 1)/m > n$, so the last payment is pulled back to $n$ and
proportionally reduced by $w$.

## The fractional weight

The core quantity is the fractional part of $n \cdot m$:

$$w = n \cdot m - \lfloor n \cdot m \rfloor \in [0, 1)$$

When $w = 0$ the term is exactly aligned with the payment grid and no adjustment is needed.
When $w > 0$ the final period is incomplete and $w$ is used as the scaling weight.

:::{note}
When a time shift `ts > 0` and deferment `d` are used together, $w$ is computed from the
**effective** term $n_\text{eff} = n - \max(ts - d,\, 0)$ rather than from raw $n$.
When $ts \le d$ the shift falls entirely within the deferment period and $n_\text{eff} = n$.
:::

**Example 1 — aligned** ($n = 10.75$, $m = 4$, quarterly):

$$nm = 10.75 \times 4 = 43, \qquad w = 43 - 43 = 0$$

No fractional adjustment is applied.

**Example 2 — incomplete** ($n = 10.6$, $m = 4$, quarterly):

$$nm = 10.6 \times 4 = 42.4, \qquad w = 42.4 - \lfloor 42.4 \rfloor = 0.4$$

The last quarter covers only $w/m = 0.1$ year out of a full sub-period of $1/m = 0.25$ years.

## Annuity adjustment

For `ax`, `axy`, `axyz`, `ajoint` (annuity-immediate) with $w > 0$:

- The **last payment time** is the exact term endpoint $n_\text{eff} + d$ (where $d$ is
  the deferment period; for the common case $d = 0$ this is simply $n_\text{eff}$).
- The **last payment amount** is scaled by $w$:

$$\text{PV contribution}_{\text{last}} = \frac{w}{m} \cdot {}_{n_\text{eff}}p_x \cdot v^{n_\text{eff}+d} \cdot g(n_\text{eff}+d)$$

where $g(\cdot)$ is the growth factor and ${}_{n_\text{eff}}p_x$ is the survival probability
from the shifted age to $n_\text{eff}$ (for multi-life methods `axy`, `axyz`, `ajoint`,
this is the joint survival probability of all lives).

This is exposed in the `payment_adjustment` array returned by `return_flows=True`
(see {doc}`inspecting_cashflows`): all elements are 1.0 except the last, which equals $w$.

For `äx`, `äxy`, `äxyz`, `äjoint` (annuity-due), $w$ does not produce a scaled final payment:
the last payment was already made at $\lfloor nm \rfloor / m$ with full weight.

## Insurance adjustment

For all insurance methods — `Ax`, `Axy`, `Axyz`, `Afirst`, and their term and deferred
variants — the fractional-period scaling applies whenever $w > 0$.  Insurances have no
due/immediate distinction (that concept is replaced by `mortality_placement`), so the
adjustment fires regardless of any annuity-related convention:

- The **last period's death probability** is scaled by $w$:

$${}_{1/m}q_{x + t_{\text{last}}}^{\text{adjusted}} = w \cdot {}_{1/m}q_{x + t_{\text{last}}}$$

- The **last period's discount time** offset is also scaled:

$$t_{\text{discount,\,last}} = t_{\text{last}} + \delta_m \cdot w$$

where $\delta_m$ is the `mortality_placement` offset ($0$, $1/(2m)$, or $1/m$).

Both effects are visible in the `return_flows=True` dict via the
`death_probability_adjustment` and `discount_time` arrays — see {doc}`inspecting_cashflows`.

## Irregular cashflows

When using `cashflow_times` and `cashflow_amounts` instead of the `n`/`m` grid, the
fractional adjustment is **not applied** — the programmer controls each payment's timing
and amount directly.  See {doc}`irregular_cashflows` for details.

:::{note}
`äjoint` does not accept `cashflow_times` (annuity-due with multiple lives uses the
standard grid only).  For irregular schedules with multiple lives use `ajoint`, `axy`,
`axyz`, or `ax` with explicit `cashflow_times`.
:::

## See also

- {doc}`inspecting_cashflows` — `payment_adjustment` and `death_probability_adjustment` arrays
- {doc}`calculation_modes` — `discrete_precision` is the only mode that applies this adjustment
- {doc}`irregular_cashflows` — arbitrary cashflow timing
- {doc}`configuration` — `force_integer_ts` and other calculation settings
- {doc}`notation_glossary` — $\ddot{a}$, $a$, $m$, $n$ symbol definitions

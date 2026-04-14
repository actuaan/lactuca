# Force of Mortality Methods

The **force of mortality** $\mu_x$ is the instantaneous rate at which a life aged $x$ is dying.
It is defined as:

$$\mu_x = -\frac{\mathrm{d}}{\mathrm{d}x} \ln l_x = \frac{f_x}{S(x)}$$

where $f_x$ is the age-at-death density and $S(x) = l_x / l_0$ is the survival function.

Lactuca uses $\mu_x$ in **{doc}`continuous calculation modes <calculation_modes>`** (`continuous_precision`,
`continuous_simplified`) to evaluate the integrand of life annuity and insurance formulas.

## Available methods

The method is set via `config.force_mortality_method`:

| Method | Approach | SciPy required | Best for |
|--------|----------|----------------|----------|
| `"finite_difference"` (default) | Central log-differences | No | Standard integer-age tables |
| `"spline"` | Natural cubic spline on $\ln({}_{t}p_x)$ | Yes | Smooth graduated tables |
| `"kernel"` | Gaussian kernel smoothing | Yes | Noisy or empirical data |

```python
from lactuca import config

config.force_mortality_method = "finite_difference"  # default
config.force_mortality_method = "spline"
config.force_mortality_method = "kernel"
config.reset()
```

### `finite_difference` (default)

Applies central finite differences to the **log-survival** function $\ln({}_{t}p_x)$:

$$\mu_i \approx -\frac{\ln({}_{t_{i+1}}p_x) - \ln({}_{t_{i-1}}p_x)}{t_{i+1} - t_{i-1}}
\quad \text{(interior points)}$$

For a standard life-table grid at integer ages ($t_{i\pm1} = t_i \pm 1$, unit spacing),
this reduces to the well-known log-central-difference formula:

$$\mu_j \approx \frac{\ln l_{j-1} - \ln l_{j+1}}{2}
= \frac{1}{2}\ln\frac{l_{j-1}}{l_{j+1}}$$

At the first and last grid points, one-sided log-differences are used instead.
This is the fastest and most common method for standard tabular life table work.

### `spline`

Fits a **cubic spline** to $\ln({}_{t}p_x)$ as a function of $t$, then evaluates
the analytical derivative of that spline:

$$\mu(t) = -\frac{\mathrm{d}}{\mathrm{d}t}\,\mathcal{S}(t)$$

where $\mathcal{S}$ is the natural cubic spline through the log-survival points.
Provides $C^2$-continuous (twice continuously differentiable) force estimates.
Best suited for **smooth graduated tables** where continuity of the derivative
is important.  Requires SciPy (`scipy.interpolate.CubicSpline`, loaded lazily).

### `kernel`

Applies **Gaussian kernel smoothing** to $\ln({}_{t}p_x)$ and then computes the
numerical gradient:

$$\mu(t) \approx -\frac{\mathrm{d}}{\mathrm{d}t}\bigl[G_\sigma * \ln({}_{t}p_x)\bigr]$$

where $G_\sigma$ is a Gaussian kernel with bandwidth $\sigma = 1.0$ grid-point units (fixed).
Provides robust smoothing without global parametric assumptions.
Best suited for **noisy or irregular empirical data** (small populations, raw
survey-based tables).  Requires SciPy (`scipy.ndimage.gaussian_filter1d`, loaded lazily).

## Mortality placement

In addition to `force_mortality_method`, `config.mortality_placement` controls the timing
of death benefit payments within each sub-annual period (`"beginning"`, `"mid"` (default),
`"end"`).  It affects **insurance calculations in all modes** and is independent of
`force_mortality_method`.  For the full reference including the $\delta_m$ offset values
and code example, see {ref}`mortality placement <mortality-placement>` in {doc}`calculation_modes`.

## Choosing a method

All three `force_mortality_method` values are mathematically compatible with both
`lx_interpolation` settings.  The choice is primarily about numerical behaviour:

| `lx_interpolation` | Recommended `force_mortality_method` | Notes |
|--------------------|--------------------------------------|-------|
| `"linear"` (UDD) | `"finite_difference"` | Fastest; sufficient accuracy for standard tables |
| `"exponential"` (CFM) | `"finite_difference"` | Consistent with constant-force shape; most common choice |

`"spline"` and `"kernel"` are usable with either interpolation setting and are preferred
when the computed force of mortality must be smooth (e.g. for gradation checks or
presentation) or when the underlying table is noisy.

Mixing any `force_mortality_method` with any `lx_interpolation` is allowed; minor
numerical differences between discrete and continuous results may arise from the
different smoothness assumptions.

:::{note}
`force_mortality_method` applies **only to continuous integration modes**.  For discrete
modes, survival probabilities are computed directly from the $l_x$ array via interpolation
and $\mu_x$ is not evaluated.
:::

## See also

- {doc}`lx_interpolation` — fractional-age assumptions for $l_{x+s}$
- {doc}`calculation_modes` — when continuous modes are used
- {doc}`numerical_precision` — precision of unrounded $l_x$ values used in continuous integration
- {doc}`configuration` — full settings reference

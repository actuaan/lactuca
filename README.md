# Lactuca

**Professional actuarial library for Python** — life insurance, pensions, and disability models: mortality and disability tables, annuities, insurances, commutation functions, and interest rate scenarios, optimized for Spanish and international actuarial practice.

[![PyPI](https://img.shields.io/pypi/v/lactuca)](https://pypi.org/project/lactuca/)
[![Python](https://img.shields.io/pypi/pyversions/lactuca)](https://pypi.org/project/lactuca/)
[![License](https://img.shields.io/badge/license-Proprietary-red)](https://github.com/actuaan/lactuca/blob/main/LICENSE)
[![Docs](https://img.shields.io/badge/docs-latest-blue)](https://www.lactuca.io/latest/)

## Installation

```bash
pip install lactuca
```

Requires **Python ≥ 3.12**. Core runtime dependencies include NumPy, SciPy, Pandas, Polars, msgpack, PyNaCl, requests, and platformdirs (see `pyproject.toml` for pinned ranges).

## Activation

**Proprietary software** — a valid license key is required before calculations run.

On the first use (`python -m lactuca` or `import lactuca`), the library looks for a license in this order:

1. **`LACTUCA_LICENSE_KEY`** environment variable (recommended for servers and CI/CD).
2. Local **`license.json`** in your Lactuca config directory (offline use after first activation).
3. **Interactive prompt** — enter an existing key or request a free 30-day trial.

```bash
# Interactive activation (same as omitting the subcommand)
python -m lactuca
python -m lactuca activate

# License maintenance (script-friendly)
python -m lactuca license status [--json]
python -m lactuca license refresh [--json]
python -m lactuca license doctor [--json]
```

- 🆓 **Free trial**: [www.lactuca.io/pricing](https://www.lactuca.io/pricing)
- 💼 **Buy a license**: [www.lactuca.io/pricing](https://www.lactuca.io/pricing)
- 📄 **License terms**: [LICENSE](https://github.com/actuaan/lactuca/blob/main/LICENSE)
- 📖 **Full activation guide**: [Activation Guide](https://www.lactuca.io/activation)

### Check the version

```bash
python -c "import lactuca; print(lactuca.__version__)"
```

The first import runs activation (see above). To read pip metadata **without** importing the package:

```bash
pip show lactuca
python -c "from importlib.metadata import version; print(version('lactuca'))"
```

## Quick Start

```python
from lactuca import LifeTable, äx

# Load a generational mortality table (PASEM 2020, male)
lt = LifeTable("PASEM2020_Gen_2o", "m")

# 15-year temporary life annuity-due, annual payments (m=1), 3% interest
# n = term in years, m = payment frequency per year (1=annual, 12=monthly)
annuity = lt.äx(x=50, n=15, m=1, ir=0.03)
print(f"Life annuity value: {annuity:.4f}")

# Identical result via the functional API:
annuity = äx(table=lt, x=50, n=15, m=1, ir=0.03)
print(f"Life annuity value: {annuity:.4f}")
```

## Features

- ✅ **International actuarial tables** — bundled and ready to use:
  - **Spain**: PASEM2010, PASEM2020 (Gen/NoRel/Rel/Dec), PER2020 (Col/Ind)
  - **Germany**: DAV2004R (Aggregate and Select-Ultimate, 1st/2nd order)
  - **USA**: GAM71, GAM83, GAM94
  - **UK**: AM92, AF92
  - **Chile (CMF NCG 305/2023)**: CB, MI, RV, B tables
  - **Other**: IASS90, SS90
- ✅ **Generational mortality** with exponential, linear, and discrete improvement factors
- ✅ **Life, disability, and exit tables** with multiple decrements
- ✅ **Annuities** (discrete/continuous, immediate/due, fractional frequencies)
- ✅ **Life insurances** (term, whole life, endowment)
- ✅ **Commutation functions** (Dx, Nx, Cx, Mx, Rx, Sx, Lx, Tx, ex and joint-life variants)
- ✅ **Functional API** (scalar and vectorized: `ax`, `äx`, `Ax`, `tpx`, `ex`, `nEx`, joint-life `axy`, `Axy`…)
- ✅ **Batch API** — vectorized multi-policy calculations with scalar equivalence when N=1
- ✅ **Flexible interest rates** with `InterestRate` (constant rate or piecewise term structure)
- ✅ **Growth rate scenarios** with `GrowthRate` for benefit escalation
- ✅ **Fractional time shifts** (ts) with UDD/CFM interpolation methods
- ✅ **Actuarial date utilities** (exact age, last/nearest/next birthday, anniversary dates)
- ✅ **Highly optimized** with NumPy vectorization and Cython compilation
- ✅ **Type-safe** with full type hint coverage and stub files (.pyi)

## Documentation

📚 **[Full Documentation](https://www.lactuca.io/latest/)**

- [Getting Started Guide](https://www.lactuca.io/latest/user_guide/getting_started.html)
- [API Reference](https://www.lactuca.io/latest/api/)
- [Actuarial Formulas](https://www.lactuca.io/latest/formulas.html)
- [Changelog](https://www.lactuca.io/latest/changelog.html)

Documentation is licensed under [CC BY 4.0](https://github.com/actuaan/lactuca/blob/main/LICENSE_DOCS). The library code and binaries remain proprietary — see [LICENSE](https://github.com/actuaan/lactuca/blob/main/LICENSE).

## Contact

📧 [support@lactuca.io](mailto:support@lactuca.io)  
🌐 [https://www.lactuca.io](https://www.lactuca.io)

---

<sub>Built with ❤️ for the actuarial community using Cython for maximum performance.</sub>

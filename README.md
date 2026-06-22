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

Requires **Python ≥ 3.12**. Runs on **Windows, Linux, and macOS** (wheels for CPython 3.12–3.14). Core runtime dependencies include NumPy, SciPy, Pandas, Polars, msgpack, PyNaCl, requests, and platformdirs (see `pyproject.toml` for pinned ranges).

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

> **Requires an activated license** — see [Activation](#activation) above (`LACTUCA_LICENSE_KEY`, `license.json`, or the interactive trial).

Lactuca centres on three table classes — `LifeTable` (mortality), `DisabilityTable` (disability incidence), and `ExitTable` (withdrawals) — plus `InterestRate`, `GrowthRate`, and a global `Config` for calculation defaults. **Generational** tables require a `cohort` birth year; **select-ultimate** tables require a `duration` (select year, or `"ult"` for ultimate); some tables need both.

```python
from lactuca import LifeTable, äx

# Generational mortality table (PASEM 2020, male)
lt = LifeTable("PASEM2020_Gen_2o", "m")

# First-order individual table with explicit cohort (PER 2020, male, born 1969)
lt_cohort = LifeTable("PER2020_Ind_1o", "m", cohort=1969)

# Select-ultimate table (UK AM92/AF92, male) — duration picks the select slice (1 = first select year)
lt_select = LifeTable("AM92_AF92", "m", duration=1)

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
  - **Spain (DGSFP)**: PASEM2010, PASEM2020 (Gen/NoRel/Rel/Dec), PER2020 (Col/Ind), PEAI2007
  - **Spain (SS)**: IASS90, SS90
  - **Germany (DAV)**: DAV2004R (Aggregate and Select-Ultimate, 1st/2nd order)
  - **USA (SOA)**: GAM71, GAM83, GAM94
  - **UK (CMIB)**: AM92_AF92
  - **Switzerland (SOA)**: GRMF80, GRMF95, GKMF80, GKMF95
  - **Chile (CMF NCG 305/2023)**: CB, MI, RV, B tables
- ✅ **Generational mortality** with exponential, linear, and discrete improvement factors (`cohort`); **select-ultimate** tables with duration-dependent rates (`duration` or `"ult"`)
- ✅ **`LifeTable`**, **`DisabilityTable`**, and **`ExitTable`** — mortality, disability incidence, and exit/withdrawal decrements (including multiple-decrement combinations)
- ✅ **Custom tables** — build proprietary `.ltk` tables with `TableBuilder` or load them via `read_table`
- ✅ **Annuities** (discrete/continuous, immediate/due, fractional frequencies)
- ✅ **Life insurances** (term, whole life, endowment)
- ✅ **Commutation functions** (Dx, Nx, Cx, Mx, Rx, Sx, Lx, Tx, ex and joint-life variants)
- ✅ **Functional API** (scalar and vectorized: `ax`, `äx`, `Ax`, `tpx`, `ex`, `nEx`, joint-life `axy`, `Axy`, first-death `Afirst`, N-life `ajoint`…)
- ✅ **Batch API** — vectorized multi-policy calculations with scalar equivalence when N=1
- ✅ **Flexible interest rates** with `InterestRate` (flat rate, piecewise term structure, named scenarios)
- ✅ **Growth rate scenarios** with `GrowthRate` for benefit escalation
- ✅ **Global `Config`** — calculation modes (`discrete_precision`, `continuous_precision`, …), mortality placement, and interpolation defaults
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

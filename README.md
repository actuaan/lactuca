# Lactuca

**Professional actuarial library for Python** — life tables, disability models, annuities, and interest rate scenarios, optimized for Spanish and international actuarial practice.

[![PyPI](https://img.shields.io/pypi/v/lactuca)](https://pypi.org/project/lactuca/)
[![Python](https://img.shields.io/pypi/pyversions/lactuca)](https://pypi.org/project/lactuca/)
[![License](https://img.shields.io/badge/license-Proprietary-red)](LICENSE)
[![Docs](https://img.shields.io/badge/docs-latest-blue)](https://actuaan.github.io/lactuca/latest/)

## Installation

```bash
pip install lactuca
```

## Quick Start

```python
from lactuca import LifeTable, InterestRate

# Load Spanish actuarial table
table = LifeTable("PASEM2020_Gen_2o", "m")

# Calculate life annuity (annual payments, 3% interest)
ir = InterestRate(0.03)
annuity = table.äx(age=50, n=15, m=1, ir=ir)
print(f"Life annuity value: {annuity:.4f}")
```

## Features

- ✅ **Spanish actuarial tables** (PASEM2010, PASEM2020, PER2020, DAV 2004 R)
- ✅ **International tables** (CMI 2021, SOA MP-2021, DAV 2008 T, and more)
- ✅ **Generational mortality** with exponential/linear improvements
- ✅ **Life, disability, and exit tables** with multiple decrements
- ✅ **Annuities** (discrete/continuous, immediate/due, fractional frequencies)
- ✅ **Life insurances** (term, whole life, endowment)
- ✅ **Flexible interest rates** (constant/piecewise term structures)
- ✅ **Growth rate scenarios** with `Growth` class for benefit escalation
- ✅ **Fractional time shifts** (ts) with UDD/CFM interpolation methods
- ✅ **Actuarial date utilities** (exact age calculation, anniversary dates)
- ✅ **Highly optimized** with NumPy vectorization and Cython compilation
- ✅ **Type-safe** with full type hint coverage and stub files (.pyi)

## Documentation

📚 **[Full Documentation](https://actuaan.github.io/lactuca/latest/)**

- [Getting Started Guide](https://actuaan.github.io/lactuca/latest/getting_started.html)
- [API Reference](https://actuaan.github.io/lactuca/latest/api/)
- [Actuarial Formulas](https://actuaan.github.io/lactuca/latest/formulas.html)
- [Changelog](https://actuaan.github.io/lactuca/latest/changelog.html)

## Requirements

- **Python** ≥ 3.10
- **Dependencies**: NumPy ≥ 2.3, Pandas ≥ 2.3, Polars ≥ 1.34, SciPy ≥ 1.16, msgpack, tomli

## License

**Proprietary software** — See [LICENSE](LICENSE) for commercial licensing terms.

**Documentation** licensed under [CC BY 4.0](LICENSE_DOCS) — You are free to share and adapt the documentation with attribution.

## Contact

**Alberto Aragoneses Nebreda**
📧 [alberto.aragoneses@actuaan.com](mailto:alberto.aragoneses@actuaan.com)
🌐 [https://actuaan.github.io/lactuca/latest/](https://actuaan.github.io/lactuca/latest/)

---

<sub>Built with ❤️ for the actuarial community using Cython for maximum performance.</sub>

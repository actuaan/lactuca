---
html_theme.sidebar_secondary.remove: true
---

# Lactuca Documentation

**Life actuarial calculations for Python — precise, fast, production-ready**

Lactuca is a high-performance library for **life actuarial calculations**, consistent with international actuarial standards.
It covers life contingencies, insurances, annuities, multiple-decrement models, and interest rate scenarios —
vectorised with NumPy for production performance.

::::{grid} 1 1 2 2
:gutter: 3

:::{grid-item-card} {octicon}`rocket;1.5em` Quick Start
:link: user_guide/getting_started
:link-type: doc

Get started with Lactuca in minutes. Load actuarial tables, calculate life annuities, and explore the core API.
:::

:::{grid-item-card} {octicon}`book;1.5em` User Guide
:link: user_guide/index
:link-type: doc

Step-by-step guides covering tables, calculation modes, interest rates, joint-life calculations, off-anniversary reserves, and more.
:::

:::{grid-item-card} {octicon}`cpu;1.5em` AI Assistants
:link: user_guide/using_ai_assistants
:link-type: doc

Context files and workflows for Cursor, Copilot, ChatGPT, and other AI tools.
:::

:::{grid-item-card} {octicon}`beaker;1.5em` Cookbook
:link: cookbook
:link-type: doc

Copy-paste actuarial recipes for common valuation tasks.
:::

:::{grid-item-card} {octicon}`mortar-board;1.5em` Actuarial Formulas
:link: formulas
:link-type: doc

Mathematical foundations and actuarial formulas used throughout the library, with LaTeX notation.
:::

:::{grid-item-card} {octicon}`list-unordered;1.5em` API Reference
:link: api/index
:link-type: doc

Complete API documentation for all public classes, functions, and parameters.
:::

::::

## Features

**Tables**

- ✅ **Aggregate, select-ultimate, static and generational tables** — all major table structures supported
- ✅ **Ready-to-use bundled tables** from Spain, Germany, Chile, USA, UK, Switzerland, and more — see the [complete list](user_guide/bundled_tables)
- ✅ **Generational mortality** with exponential, linear, discrete, and projected improvement formulas

**Calculations**

- ✅ **Multiple-decrement models** — life, disability, and exit tables combinable via `table_combination`
- ✅ **Annuities** (discrete/continuous, immediate/due, fractional frequencies)
- ✅ **Life insurances** (term, whole life, endowment)
- ✅ **Flexible interest rates** with `InterestRate` (constant, piecewise, and named scenarios)
- ✅ **Growth rate scenarios** with `GrowthRate` class for benefit escalation
- ✅ **Fractional time shifts** (`ts`) with configurable integer-enforcement policy
- ✅ **Actuarial date utilities** (exact age calculation, anniversary dates)

**Platform**

- ✅ **Highly optimised** with NumPy vectorisation for production workloads
- ✅ **IDE-friendly**: full autocompletion in Jupyter and VS Code via stub files (.pyi)

```{toctree}
:maxdepth: 2
:hidden:
:caption: User Guide

user_guide/index
```

```{toctree}
:maxdepth: 2
:hidden:
:caption: Reference

api/index
formulas
cookbook
errors_reference
```

```{toctree}
:maxdepth: 1
:hidden:
:caption: Licensing

pricing
activation
faq_licensing
eula
eula_es
privacy
privacy_es
```

```{toctree}
:maxdepth: 1
:hidden:
:caption: Project

citing
support
changelog
```

## Indices

- {ref}`genindex`
- {ref}`search`

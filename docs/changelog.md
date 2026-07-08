# Changelog

All notable changes to Lactuca will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.1.11] - 2026-07-08

### Features

- **tables**: TBL-01 deferred instantiation with pending=True and configure()

- **tables**: TBL-01 exports, tests P-01..P-27, and setter/warning fixes


### Bug Fixes

- **_activation**: Auto-reclaim stale process seats before LAC-4001

- **_activation,docs**: Align LAC-4001 message and user docs with auto-recovery

- **ci**: Own gh-pages root landing page on prod docs deploy

- **scripts,ci**: Generate changelog per prod tag, drop emojis

- **tables**: Accept cartesian/return_dict in concrete table __init__

- **tables**: Close TBL-01 audit gaps for pending configure

- **tables**: Modernize multi-exception except in decrement.py

- **tables**: Widen _apply_and_rebuild params for wheel sentinel

- **tables**: Widen configure() sentinel locals for wheel

- **tables**: Keyword-only configure rebuild for Cython wheels

- **tables,tests**: Select duration flag and pending configure gaps

- **scripts,tests**: Enrich constructor stubs regardless of source Union

- **tables**: Widen TableRegistry table_class for Cython wheels

- **tables**: Harden TableRegistry table_class validation for wheels

- **ci**: Copy src and extra scripts into isolated wheel pytest root

- **ci**: Align prod release gates with test workflow


### Documentation

- **_activation,exceptions**: Complete LIC-01 LR-9/LR-12/LR-10b

- **tables**: TBL-01 docstrings for configure, configure_all, batch_update, TableRegistry

- **tables**: TBL-01 docstrings for constructors, setters, qx/px/view_data

- **tables**: TBL-01 docstrings LifeTable constructors and summary

- **docs**: TBL-01 pending configure user guide and API pages

- **tables**: Polish TBL-01 pending docstrings after audit

- **docs**: Fix errors_reference cross-ref in using_tables

- **docs,ci**: Add sitemap, canonical URLs, and robots.txt for SEO

- **docs**: Document pending unisex configure in using_tables

- **mkt**: Add Spanish demo video artifacts for FASE_0 launch

- **docs**: Clarify TableRegistry requires concrete table class

- **mkt**: Relocate Spanish demo notebook to notebooks/

- **mkt**: Add executed outputs to lactuca_demo_es notebook

- **mkt**: Fix actuarial coherence in lactuca_demo_es notebook

- **mkt**: Use reset_to_defaults in lactuca_demo_es section 5


### Testing

- **tests**: Fix TBL-01 suite regressions and sync gate docs


### Miscellaneous

- **scripts**: Remove verify_pending_docs harness

## [0.1.6] - 2026-07-05

### Features

- **_activation**: Add license release-stale and release --force CLI


### Bug Fixes

- **scripts**: Enrich wheel pyi stubs for IDE typing contract (IDE-01)

- **tests**: Read wheel pyi in test_64 when scripts tree absent

- **scripts**: Resolve re-exported Literal aliases in wheel pyi stubs

- **scripts,tests**: Harden wheel stub IDE contract after import audit

- **scripts**: Emit local Literal aliases without duplicate imports

- **scripts,tests**: Expand Tier 1 stub overrides and union ordering contract

- **ci**: Use wheel artifacts for Test PyPI stub validation

- **ci**: Stage docs scripts for isolated wheel pytest

- **ci**: Run wheel E2E steps with bash shell on Windows


### Performance

- **scripts**: Public-only pyi stubs and faster stub generator


### Refactoring

- **api**: Import PaymentFrequencyLiteral from lactuca.base


### Documentation

- **changelog**: Exclude test PyPI tags from public history

- **engine,ci**: Clarify engine base re-exports and scripts-utilities Track 2c

- **activation**: Document license release-stale and release --force CLI

- **conf**: Move version-switcher between theme and github icons


### CI/CD

- **docs**: Use committed changelog in doc workflows

- **docs,scripts,tests**: Enable version switcher and accumulative gh-pages deploy (DOCS-01)


### Miscellaneous

- **build**: Set PyPI classifier to Production/Stable

## [0.1.0] - 2026-07-02

### Features

- **utils,docs,tests**: Align cashflow utilities across code and AI docs


### Bug Fixes

- **tables**: Replace horizontal pl.concat with insert_column

- **ci**: Select wheel ABI matching E2E Python interpreter


### Documentation

- **legal**: Set EULA and privacy headers to v1.0 for production release

- **legal**: Correct EULA and privacy effective dates to 2026-07-02

- **ai**: Add dates API to AI context pack and agent rules

- **api**: Align first-death and pure-endowment terminology

- Align LICENSE with CC BY carve-out and EULA reference


### Testing

- **tests**: Relax test_56 golden ULP tolerance for wheel_test


### CI/CD

- **ci**: Avoid duplicate pytest on release push

- **ci**: Disable pre-push pytest by default

- **tests**: Fail pytest when runtime deps lag behind CI floors

- Harden wheel license smoke and align release workflows

- Fix wheel numerical smoke with table interest_rate

- Add linux canary scope and fix wheel pytest layout

- Complete wheel pytest layout and fix polars date deprecation

- Unify wheel E2E across release-test and production

- Copy swiss SOA fixture into isolated wheel pytest root
<!-- generated by git-cliff -->

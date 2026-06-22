## [Unreleased]

### ♻️ Refactoring

- **helpers**: Dedupe n-life batch helpers and harden IR/GR paths

- **ir**: Harden wheel typing and dedupe annuity batch entry


### ✅ Testing

- **tests**: Cover I-M1 machine resolution and CLI sync paths

- **tests**: Cover active_scenario resolution for ir/gr


### ✨ Features

- **batch**: Map None to np.inf in per-policy n vectors

- **batch**: Map missing n in Pandas/Polars Series to whole-life


### 🐛 Bug Fixes

- **_activation**: Resolve device machine_id by fingerprint (I-M1)

- **build**: Point PyPI Repository URL to public actuaan/lactuca

- **validators**: Harden range guards and gr arithmetic boundary


### 📚 Documentation

- **docs**: Close K2-Keygen N>1 and MPE live run 27898713330

- **docs**: Close K2-Keygen trial via B3 and add ops guide §12.6

- **docs**: Mark K5 Vercel and T3 trial webhook ops complete

- **docs**: Sync IMPL MPE header to rev 19 (K5/T3)

- **docs**: Update T-02 runbook for serial E2E and LAC-1014 retries

- **docs**: Close 9.D-9 combination_mode plan rev 4.2

- **docs**: Archive MPE and sync CI/CD plan to v0.0.30-test

- **docs**: Close F.8, 9.H Gate #29 and update licensing plan

- **readme**: Enrich PyPI landing with license notice and table params

- **docs**: Add pre-Fase 10 chain §9.R and post-prod §10.4

- **docs**: Document active_scenario call-time semantics for ir/gr

- **docs**: Expand cookbook with 16 verified actuarial recipes


### 🔧 CI/CD

- **ci**: Serialize E2E license matrix and retry LAC-1014 setup

- **sync-public**: Re-enable push trigger and fix versions.json URLs


## [0.0.31-test] - 2026-06-21

### 🐛 Bug Fixes

- **tests**: Isolate MPE CLI smoke from wheel_test license env


### 🧹 Miscellaneous

- Bump version to 0.0.31 [test release]


## [0.0.30-test] - 2026-06-21

### ⚡ Performance

- **_activation**: Derive process heartbeat interval from Keygen


### ✅ Testing

- **tests**: Parametrize MPE enforcement across license tiers

- **tests**: Add activation security hardening regression tests

- **tests**: Add TestActivationSecurityHardening regression suite

- **tests**: Add Keygen MPE CLI smoke tests without live skip


### ✨ Features

- **ci**: Add LIVE-4 Team and LIVE-5 OEM MPE live scenarios

- **ci**: Add LIVE-4b enterprise pool scenario to MPE Keygen live


### 🐛 Bug Fixes

- **ci**: Install build deps before MPE live editable install

- **ci**: Paginate Keygen machine list in MPE live script

- **_activation**: Map pool-full FPS_MISMATCH to LAC-1013 on activate

- **ci**: Improve MPE live runner cleanup and lactuca fingerprint

- **_activation**: Harden pool-full FPS_MISMATCH activation mapping

- **_activation**: Infer tier from Keygen policy id for pool checks

- **ci**: Make MPE live pool seeding capacity-aware

- **_activation**: Map MACHINE_LIMIT_EXCEEDED to LAC-1013 on activate

- **_activation**: Reject foreign machine_id on VALID pool-full activate

- **_activation**: Use Keygen account processes API for LAC-4001

- **_activation**: Map MACHINE_PROCESS_LIMIT_EXCEEDED to LAC-4001

- **_activation**: Resolve OEM tier from full Keygen policy UUID

- **_activation**: Prefer OEM policy over metadata tier individual

- **_activation**: Use policy-aware tier on revalidation and CLI sync

- **_activation**: Sync OEM tier to license.json before process lease

- **_activation**: Enforce policy-authoritative tier on every import

- **_activation**: Resolve tier from server on FPS_MISMATCH pool check

- **_activation**: Enforce activation gate and block docs-build on wheels

- **ci**: Skip MPE Keygen live scenarios on API quota exhaustion

- **_activation**: Skip LAC-4003 offline grace for OEM tiers


### 📚 Documentation

- Close T5-G3 PoC on v0.0.29-test

- Add MPE CI tier matrix implementation plan

- **docs**: Document MPE cross-device errors and device registration

- **_activation**: Document MPE pool and cross-device in docstrings

- Close MPE plan and record tier-matrix embudo order

- **docs**: Document LAC-1013/1014/1015 first-activation paths

- **IMPL**: Update MPE tier matrix documentation with new specifications and affected files

- **IMPL**: Update MPE tier matrix with license expiry details

- Update keygen licenses configuration with new UUIDs

- **docs**: Close MPE tier matrix B2 and mark T-01 complete

- **docs**: Record post-B2 hardening commit SHAs in tier matrix

- **docs**: Close MPE tier matrix and distribution licensing Fase 5


### 🔧 CI/CD

- **ci**: Add MPE Keygen live workflow and helper script

- **ci**: Reuse LACTUCA_INTERNAL_LICENSE in MPE live workflow

- **ci**: Wire LIVE-2 to LACTUCA_MPE_LICENSE_TRIAL

- **ci**: Wire LIVE-2 to Individual Monthly license for LAC-4001


### 🧹 Miscellaneous

- Bump version to 0.0.30 [test release]


## [0.0.29-test] - 2026-06-19

### 🐛 Bug Fixes

- **_activation**: Map FPS_MISMATCH pool full to LAC-1013 on import


### 📚 Documentation

- Record MPE PoC T5 run 2 and T5-G3 fix status

- Add MPE PoC tier equivalence note (trial vs other tiers)


### 🧹 Miscellaneous

- Bump version to 0.0.29 [test release]

- Bump version to 0.0.29 [test release]


## [0.0.28-test] - 2026-06-19

### 🐛 Bug Fixes

- **_activation**: Handle NO_MACHINE in R4-import repair


### 📚 Documentation

- Close MPE phase 8 and add PoC T5 round-trip checklist


### 🧹 Miscellaneous

- Bump version to 0.0.28 [test release]


## [0.0.27-test] - 2026-06-19

### 🐛 Bug Fixes

- **_activation**: Auto-repair stale machine seat on import

- **_activation**: Enforce R4-import blocking server statuses


### 📚 Documentation

- Update MPE implementation status through v0.0.26-test

- Sign off gate #31 after MPE PoC v0.0.26-test

- Design R4-import repair and K2 post-PoC in MPE IMPL


### 🧹 Miscellaneous

- Bump version to 0.0.27 [test release]


## [0.0.26-test] - 2026-06-18

### ✅ Testing

- **_activation**: Add machine pool enforcement regressions


### 🐛 Bug Fixes

- **webhook**: Set metadata.tier from policy env map

- **_activation**: Enforce machine pool and cross-device rules


### 📚 Documentation

- Update implementation plan with latest CI/CD progress and metadata review

- Update implementation plan with latest CI/CD progress and release details

- **ci**: Add machine pool enforcement plan for gate #31

- **docs**: Align copy-license guidance with LAC-2002


### 🔧 CI/CD

- Fix G.6-G.8b in manual e2e workflow for recovery paths

- **ci**: Accept LAC-3002 stderr in G.8b e2e license workflow


### 🧹 Miscellaneous

- Bump version to 0.0.26 [test release]


## [0.0.25-test] - 2026-06-18

### 🐛 Bug Fixes

- **_activation,webhook**: Unify activation hints terminal-first

- **_activation**: Map NOT_FOUND to revoke and protect server metadata


### 📚 Documentation

- **ci**: Record activation message unification rev 2.102

- **ci**: Record NOT_FOUND revoke fix rev 2.103

- **ci**: Record G.10 trial E2E and Resend #5 completion rev 2.104


### 🧹 Miscellaneous

- **ci**: Disable cmake configure-on-open and orphan pyright settings

- Bump version to 0.0.25 [test release]


## [0.0.24-test] - 2026-06-17

### 🐛 Bug Fixes

- **helpers,tests**: Restore scalar finite validation in cython wheels


### 🧹 Miscellaneous

- Bump version to 0.0.24 [test release]


## [0.0.23-test] - 2026-06-17

### ✅ Testing

- **tests**: Isolate _ACTIVATION_TOKEN in CLI activate tests

- **tests**: Restore activation token after each licensed CI test

- **tests**: Clear import-time token in non-tty CLI test

- **tests**: Add non-lactuca warning policy and wheel test hygiene


### 🐛 Bug Fixes

- **helpers,tables,ir**: Silence numpy overflow and retry atomic save on windows


### 📚 Documentation

- Migrate public URLs to www.lactuca.io and update CI plan rev 2.99

- **ci**: Mark wheel_test green for v0.0.22-test (rev 2.100)

- **ci**: Update IMPL plan rev 2.101 for warning policy and wheel fixes


### 🧹 Miscellaneous

- **pyproject**: Update basedpyright configuration for tests and webhook environments

- **build**: Disable basedpyright IDE type checking

- **build**: Silence basedpyright IDE diagnostics

- Bump version to 0.0.23 [test release]


## [0.0.22-test] - 2026-06-17

### 🐛 Bug Fixes

- **_activation**: Consolidate activate CLI and suppress duplicate expiry


### 📚 Documentation

- **activation**: Document import vs CLI messaging and activate flow


### 🧹 Miscellaneous

- **skills**: Sync activation audit reference with CLI changes

- Bump version to 0.0.22 [test release]


## [0.0.21-test] - 2026-06-16

### 🐛 Bug Fixes

- **tests**: Use Python 3 tuple syntax in TableBuilder except handlers

- **tests**: Align generational qx references with qx[omega]=1.0 contract


### 🧹 Miscellaneous

- Bump version to 0.0.21 [test release]


## [0.0.20-test] - 2026-06-16

### 🐛 Bug Fixes

- **tables**: Use Python 3 tuple syntax in unisex_blend except handlers


### 🧹 Miscellaneous

- Bump version to 0.0.20 [test release]


## [0.0.19-test] - 2026-06-16

### ♻️ Refactoring

- **base**: Tighten ScalarSequence typing and TableKey docs

- **builder**: Close 8th-pass lactuca-optimize audit

- **engine,config**: Complete engine/base 2nd-pass optimize audit

- **engine,docs**: Extract hybrid tail orchestrators

- **tables**: DRY batch ts snap and gr_repr in life (9.A.1-B P1)

- **engine**: Post-9.D optimize annuity hybrid and batch paths

- **engine**: Post-9.D optimize insurance batch_n_life and cf_matrix

- **engine**: Post-9.D optimize helpers hybrid tail pv extraction

- **tables**: Post-9.D optimize builder terminal qx index

- **tables**: Post-9.D optimize source omega bool guard


### ⚡ Performance

- **ir,helpers,life**: Cache ts shifts and vectorize het-gr batch grouping


### ✅ Testing

- **dates**: Literal validator signatures and direct unit tests

- **tests**: Regression tests for 9.D fractional n and insurance simplified

- **tests**: Regression tests for 9.D fourth pass

- **tests**: Regression tests for 9.D fifth pass

- **tests**: Regression tests for 9.D sixth pass

- **docs**: Complete 9.D-8 hybrid fractional tail plan

- **tests**: Align TableBuilder life-table fixtures with qx[omega]=1.0

- **tests**: Close 9.D-8 hybrid fractional coverage gaps

- **tests**: Add 9.D-10 age_shift table_combination alignment tests

- **tests**: Add 9.D-11 MDDT scope and key-order regression tests

- **tests**: Add TestPass17DeepAudit for 9.D-17 certainty truncation


### ✨ Features

- **tables**: Surface omega truncation after table_combination

- **tables,ir**: I-2 diagonal metadata and I-6 curve_policy [**BREAKING**]

- **tables**: Add combination_mode independent/udd for table_combination


### 🐛 Bug Fixes

- **ir,life**: Ts-shift cache epoch invalidates on curve mutation

- **config,ir**: Unify calendar validation and reset callback chain

- **tables**: Harden TableSource load validation and cache

- **tables**: Wheel-safe decrement setters, qx cache, and DRY hot paths

- **tables**: Audit-clean disability ix, DRY summary, exit parity

- **tables**: Audit-clean exit ox docstrings and blocked API

- **tables**: Audit-clean tables/base generational helpers

- **tables**: Audit-clean tables/data installer batch API

- **data**: Audit-clean data_tables CMF metadata and DAV cache

- **utils**: Wheel-safe hybrid signatures and tiered validation

- **gr**: Audit-clean arithmetic validation and DRY helpers

- **_activation**: Clarify offline revalidation UX and docstrings

- **exceptions**: Complete 2nd-pass licensing exception API

- **engine,tests**: Complete endowment 2nd-pass lactuca-optimize audit

- **dates,base,engine,helpers,tests**: Complete dates/core 6th-pass audit

- **dates**: Complete base/helpers 2nd pass and act_age validation

- **api**: Entry point audit fixes after 9.F.5

- **engine**: Shorten Ax2 term in insurance simplified modes

- **tables**: Align mi_by_duration boundary with select column routing

- **engine,tables**: 9.D actuarial corrections for simplified modes

- **engine,tables**: 9.D fourth pass batch ts and table guards

- **tables,engine,helpers**: 9.D fifth pass P0/P1 actuarial fixes

- **engine,tables**: 9.D sixth pass actuarial fixes

- **tables**: Align table_combination by age index

- **engine,docs**: Restore simplified mode independence on fractional n

- **engine,docs**: Hybrid fractional tail in discrete_simplified

- **tables**: Restore discrete Lx/Tx/ex under exponential interpolation

- **tables**: Reject negative x in tpx t=1 fast path

- **tests**: Reorder import statements and simplify assertion message in LifeTable tests

- **tables,docs,tests**: Respect stored qx at terminal age omega

- **tables,docs,tests**: Restore qx[omega]=1.0 terminal age requirement

- **tables,tests**: Wire combination_mode udd to udd2/udd3 helpers

- **tables**: Calendar-age alignment for table_combination after age_shift

- **tables**: Udd fractional mddt cache and age_shift other guard

- **tables**: Align UDD fractional survival with collapsed q comb

- **tables**: Clarify combine truncation age after age_shift (9.D-14)

- **tables**: Reject duplicate/self combine and calendar q>1 (9.D-15)

- **tables**: Validate padding and rate bounds before combine (9.D-16)

- **tables**: Truncate at incoherent certainty tail in modify pipeline (9.D-17)

- **tables**: Reject bool age_shift in modify pipeline (9.A.1-B P0)

- **webhook**: Harden OWASP audit findings for Gate 9.G

- **_activation**: Point trial webhook to production URL

- **_activation**: Type lazy requests import for pyright

- **_activation**: Fix requests cast and duplicate activation menu


### 👷 Build System

- Configure cursorpyright webhook env and vscode

- **webhook**: Add requirements.txt for vercel production

- **webhook**: Drop legacy builds from vercel.json

- Remove duplicate pyright sections from pyproject.toml


### 📚 Documentation

- **my_docs**: Record GAP-C6 wheel bench and close Gate #30

- **my_docs**: Record ir reserve batch 6th pass in CI/CD plan

- **my_docs**: Record config 6th-7th pass audit-clean in CI/CD plan

- **my_docs**: Record base.py 2nd pass audit-clean in CI/CD plan

- **my_docs**: Record builder 8th pass audit-clean in CI/CD plan

- **my_docs**: Record source.py 2nd pass audit-clean in CI/CD plan

- **my_docs**: Record decrement.py 2nd pass audit-clean in CI/CD plan

- **my_docs**: Record exit.py 2nd pass audit-clean in CI/CD plan

- **my_docs**: Record tables/base.py 2nd pass audit-clean in CI/CD plan

- **my_docs**: Record tables/data.py 2nd pass audit-clean in CI/CD plan

- **my_docs**: Record utils.py 2nd pass audit-clean in CI/CD plan

- **my_docs**: Record growth_rates.py 2nd pass audit-clean in CI/CD plan

- **my_docs**: Record _activation.py 9.F audit sign-off

- **docs**: Align errors_reference license section with exceptions API

- **my_docs**: Record exceptions.py 2nd pass sign-off in CI/CD plan

- **my_docs**: Record engine/base.py 2nd pass in CI/CD plan

- **my_docs**: Record engine/endowment.py 2nd pass in CI/CD plan

- **my_docs**: Record dates/core.py 6th pass in CI/CD plan

- **my_docs**: Record dates 26-27 2nd pass and 9.A.1 sweep complete

- **my_docs**: Record 9.F.5 entry points audit and 9.A.1 scope

- **my_docs**: Record 9.D third pass actuarial audit session

- **my_docs**: Record 9.D fourth pass and 9.E.0 P2 backlog

- **my_docs**: Record 9.D fifth pass and 9.E.0 P2 backlog

- **docs**: 9.E initial actuarial user guide updates

- **my_docs**: Record 9.D sixth pass and 9.E partial progress

- **docs**: Document table_combination age-alignment contract

- Mark 9.D-8 commit hash in hybrid fractional plan

- Unify qx omega boundary criteria across agent docs

- **my_docs**: Record G-06 commit hash in 9.D-8 plan

- **my_docs**: Record G-07 commit hash in 9.D-8 plan

- **docs,my_docs**: Close 9.D I-3 through I-6 actuarial docs

- **my_docs**: Record I-2..I-6 commit hashes in CI/CD plan

- **my_docs**: Record qx[omega] revert commit hash in CI/CD plan

- **my_docs**: Add combination_mode UDD plan rev.3 for 9.D-9

- **my_docs**: Add BACKLOG and refine combination_mode plan rev 4.1

- **my_docs**: Delegate formulas.md MDDT patch to 9.E.0

- **docs**: Table_combination UDD formulas and others-order note

- **my_docs**: Record 9.D-9 F-01 UDD hot path and E-12 closure

- **docs**: Document calendar-age alignment for age_shift and table_combination

- **my_docs**: Record 9.D-10 C-1 calendar-age alignment closure

- **tables**: Clarify v1 MDDT scope in _modify_decrement notes

- **docs**: Clarify v1 MDDT scope and modification key order

- **my_docs**: Record 9.D-11 IMPORTANT audit closure

- **docs**: 9.D-12 table_combination I-D I-E and fractional UDD

- **my_docs**: Record 9.D-12 audit closure

- **docs**: Document Bowers v1 collapsed fractional survival (9.D-13)

- **my_docs**: Record 9.D-13 audit closure and BACKLOG CM-V2-05

- **docs**: Table_combination contracts for 9.D-14 I-B/C/D

- **my_docs**: Record 9.D-14 audit closure (rev 2.68)

- **docs**: Duplicate/select-ultimate and q>1 errors for 9.D-15

- **my_docs**: Record 9.D-15 audit closure (rev 2.69)

- **docs**: Pre-combine padding and rate guards for 9.D-16

- **my_docs**: Record 9.D-16 audit closure (rev 2.70)

- **docs**: Certainty truncation and generational combine notes (9.D-17)

- **my_docs**: Record 9.D-17 audit closure (rev 2.71)

- **my_docs**: Record 9.D re-audit closure and 9.E.1 next step (rev 2.72)

- **docs**: Close IR-4D-01/04/05/06 in interest rates user guide

- **ir**: Clarify calculation_mode source and get_rate junctions

- **my_docs**: Record 9.E.1-A closure and next step 9.E.1-B (rev 2.73)

- **my_docs**: Add 9.A.1-B optimize post-9.D plan and P0 audit

- **tables**: Record 9.A.1-B P2 post-9.D audit for tables/base

- **tables**: Record 9.A.1-B P3 post-9.D audit for disability

- **tables**: Align exit module docstring with 9.D combination_mode

- **ci**: Close 9.A.1-B P3 engine/base post-9.D optimize

- **engine**: Align continuous_simplified docstrings (9D-004/005)

- **docs,engine**: Align continuous_simplified insurance return_flows (9D-005)

- **docs**: Separate lx_interpolation from mortality_placement (CFG-4D-01)

- **config**: Disambiguate lx_interpolation and mortality_placement docstrings

- **docs,ci**: Close 9.E/9.A audits and expand production e2e

- **docs**: Align activation CLI exit codes and offline grace wording

- **docs**: Document default activate CLI and corporate revalidation FAQ

- Record 9.G webhook audit and add 9.I threat model

- **docs**: Switch pricing checkout to lemon squeezy production

- **impl**: Record founding coupon draft in ls production

- **impl**: Record Bloque 0, Gate #18, and G.11 zero-charge path

- **impl**: Record G.11 E2E completion and next steps


### 🔧 CI/CD

- Close Gate #2 verify-license with explicit ActivationRequiredError check


### 🧹 Miscellaneous

- **benchmarks**: Add reserve ts batch benchmark and plan doc

- **tables**: Remove unused host_size in certainty truncation helper

- **plan**: Close 9.A.1-B P1 life.py and queue engine/annuity

- **plan**: Close 9.A.1-B P1 engine/annuity and queue insurance

- **plan**: Close 9.A.1-B P1 engine/insurance and queue helpers

- **plan**: Close 9.A.1-B P1 engine/helpers and queue builder

- **plan**: Close 9.A.1-B P2 builder and queue source

- **plan**: Close 9.A.1-B P2 source and queue base

- **plan**: Close 9.A.1-B P3 tables/exit.py post-9.D optimize

- **plan**: Close 9.A.1-B P3 engine/endowment post-9.D optimize

- **plan**: Close 9.A.1-B and add grep transversal prompt

- **plan**: Grep transversal 9.A.1 post-9.D

- **plan**: Mark 9.E.1-B 9D-004/005 closed (Gate #23 partial)

- **plan**: Mark 9.E.1-B+ inspecting_cashflows closed (rev 2.77)

- **plan**: Mark CFG-4D-01 closed, next 9.F (rev 2.78)

- **plan**: Mark 9.F Mode B sign-off complete (rev 2.85)

- **plan**: Document next operational step 9.G (rev 2.86)

- Bump version to 0.0.19 [test release]


## [0.0.18-test] - 2026-06-12

### 🐛 Bug Fixes

- **tables**: Widen _get_discount_factors x0 param to object for wheels


### 📚 Documentation

- **skills**: Correct Cython STRICT scalar coercion rules for wheels


### 🧹 Miscellaneous

- Bump version to 0.0.18 [test release]


## [0.0.17-test] - 2026-06-12

### 🐛 Bug Fixes

- **tables,helpers**: Wheel-safe object params in commutation finalize


### 🧹 Miscellaneous

- Bump version to 0.0.17 [test release]


## [0.0.16-test] - 2026-06-12

### 🐛 Bug Fixes

- **tables,ir**: Wheel-safe scalar/array returns for commutation and vn


### 🧹 Miscellaneous

- Bump version to 0.0.16 [test release]


## [0.0.15-test] - 2026-06-12

### 🐛 Bug Fixes

- **tables,helpers,functional**: Wheel-safe commutation and record_ids paths


### 🧹 Miscellaneous

- Bump version to 0.0.15 [test release]


## [0.0.14-test] - 2026-06-12

### 🐛 Bug Fixes

- **build**: Repair stub-parse syntax and add release preflight


### 🧹 Miscellaneous

- Remove deprecated Pyright configuration from pyproject.toml

- Bump version to 0.0.14 [test release]


## [0.0.13-test] - 2026-06-12

### ♻️ Refactoring

- **validators,ir,life**: Reuse coerced m from frequency validator

- **tables**: Use Union[float, None] for public unisex_blend

- **dates**: Dry sequence helpers in dates core re-audit

- **ir**: Apply 3rd-pass audit fixes to interest_rates

- **tables,helpers**: Unify batch product cores in life.py

- **functional**: DRY batch preamble and helpers import

- **functional**: DRY multi-table dispatch across 16 products

- **functional**: Unify multitable dispatch and DRY product preamble

- **helpers,life**: Add _batch_m_scalar_and_arr and fix n-life m docstrings


### ⚡ Performance

- **helpers,batch**: Add _resolve_batch_m and batch m helpers

- **life,engine**: Hoist m_val in batch paths; wheel-safe dispatch m

- **helpers,ir**: Np.integer m fast path and _resolve_batch_m in IR batch

- **helpers**: Use _M_FREQ_SET frozenset for O(1) m membership (GAP-C9)

- **batch,engine,tables**: Vectorize n-life discrete_precision fast path (GAP-C6)


### ✅ Testing

- **tests**: Remove legacy _wheel_linux_compat shims

- **tests**: Commutation bool guard and multi-table record_ids Series

- **tests**: Add m bool guard tests and wheel-safe typing tiers

- **tests**: Add batch m payment frequency benchmark (Fase E)

- **batch**: Add n-life prob matrix and fast-path parity tests (GAP-C6)


### 🐛 Bug Fixes

- **helpers**: Close optimize audit module 1/27

- **gr**: Restore start_f and migrate finiteness checks

- **engine**: Independent empty arrays in return_flows

- **tables**: Migrate finiteness checks for -ffast-math wheels

- **validators**: Use _is_finite_f64_array in batch range check

- **utils**: Use _is_finite_f64_array in tiered_amounts

- **ir**: Use _is_finite_f64_array in batch on_error mask

- **life**: Close optimize audit module 2/27

- **engine**: Close optimize audit module 3/27 annuity

- **engine**: Close optimize audit module 4/27 insurance

- **engine**: Close optimize audit module 5/27 helpers

- **functional**: Close optimize audit module 6/27

- **validators**: Close optimize audit module 7/27

- **ir**: Close optimize audit module 8/27 interest_rates

- **config**: Close optimize audit module 9/27

- **base**: Close optimize audit module 10/27

- **tables**: Close optimize re-audit module 11/27 builder

- **tables**: Close optimize audit module 12/27 source

- **tables,api,ir**: Close optimize audit 13/27 decrement

- **tables**: Close optimize audit module 14/27 disability

- **tables**: Close optimize audit module 15/27 exit

- **tables**: Close optimize audit module 16/27 base

- **tables**: Close optimize audit module 17/27 data

- **tables**: Close optimize audit module 18/27 data_tables

- **utils**: Close optimize audit module 19/27 utils

- **gr**: Close optimize audit module 20/27 growth_rates

- **gr**: Re-audit growth_rates eq/hash and terms validation

- **_activation**: Close optimize audit module 21/27 activation

- **_activation**: Harden Keygen JSON parsing on re-audit

- **exceptions**: Close optimize audit module 22/27 exceptions

- **engine**: Close optimize audit module 23/27 engine base

- **engine**: Close optimize audit module 24/27 endowment

- **dates**: Close optimize audit module 25/27 dates core

- **dates**: Materialize generic iterables in to_broadcast_list

- **dates**: Close optimize audit module 26/27 dates base

- **dates**: Close optimize audit module 27/27 dates helpers

- **ir**: Use content hash in _gr_group_key batch grouping

- **ir**: Simplify _gr_group_key catch-all to hash(gr_item)

- **config**: PathLike I/O signatures and singleton late-publish

- **config**: Normalize tables_path to str at storage boundary

- **tables**: Reject bool kwargs and widen _is_power_of_10 for LTO

- **tables**: Wheel kwargs, save hash guard, select_period DRY

- **ir**: Defensive copy on terms property and docstring cleanup

- **_activation**: Clarify seat offline grace comment scope

- **_activation**: Skip seat grace when machine_id absent

- **dates**: Close 9.A.1 pass 4-5 on core sequence broadcasting

- **config**: Harden wheel-safe setters and TOML validation

- **tables**: Close builder 9.A.1 pass 7 audit-clean

- **helpers,validators**: Harden batch m validation and DRY

- **life**: Hybrid object signatures, cache safety, and product Notes

- **functional,tables,ci**: Hybrid object typing and wheel bool guard

- **ir**: Hybrid object typing on InterestRate.a/ä and wheel smoke

- **engine,insurance,tests**: Close annuity 9.A.1 pass 2 audit

- **tables,functional**: Wheel bool guard on decrement probability APIs

- **api**: Commutation bool guard and functional multitable refactor

- **helpers,api**: Detect active growth rate in batch ts guard

- **functional,tests**: Align batch guards with OOP and docstring Raises

- **functional,tests**: Cython-safe multitable dispatch and shared docstrings

- **functional,tests**: Shared docstrings, bool guards, and utf8 hygiene

- **validators,helpers**: Close 7/27 audit with m fast-path

- **validators,life**: Include method name in boolean n rejection errors


### 📚 Documentation

- **ci,skills**: Record v0.0.12-test strict wheel_test and PyObject finiteness

- **ci**: Mark section 9.B.8 legacy cleanup complete (rev. 2.2)

- **api**: Align public m annotations with PaymentFrequencyLiteral

- **plan**: Sync 9.A.1 pass2 checklist rev. 2.27

- **plan**: Sync 9.A.1 builder 11/27 closure rev. 2.29

- **plan**: Sync 9.A.1 interest_rates 8/27 closure rev. 2.31

- **docs**: Record partial 9.F activation audit in CI plan

- **plan**: Record config 9.A.1 passes 4-5 audit-clean

- **plan**: Record builder 9.A.1 pass 7 audit-clean

- **plan**: Record helpers 9.A.1 pass 2 audit-clean

- **plan**: Record life.py 9.A.1 pass 2 audit-clean

- **plan**: Record annuity.py 9.A.1 pass 2 audit-clean

- **engine**: Complete insurance.py 9.A.1 pass 2 audit

- **plan**: Record insurance.py 9.A.1 pass 2 audit-clean

- **engine**: Close engine/helpers 2nd optimize pass

- **docs**: Forbid implementation leaks in user-facing documentation

- **my_docs**: Mark functional 6/27 2nd pass complete

- **my_docs,skills**: Mark validators 7/27 2nd pass complete

- **my_docs,skills**: Add batch m perf plan and retire fastpath IMPL

- **my_docs**: Mark Fase F items 20-21 complete in batch m perf plan

- **my_docs**: Mark Fase E benchmark complete in batch m perf plan

- **my_docs,skills**: Close batch m perf Fase C documentation

- **my_docs**: Add GAP-C6 n-life batch vectorization plan

- **my_docs**: Integrate GAP-C6 into CI/CD plan and mark Fase 2 complete

- **my_docs**: Mark GAP-C6 merge done and clarify Fase 3 scope


### 🧹 Miscellaneous

- **build**: Remove pyright from dev tooling

- **docs**: Drop pyright from copilot cython instructions

- **plan**: 9.A.1 pass2 module 13/27 clean

- Bump version to 0.0.13 [test release]


## [0.0.12-test] - 2026-06-08

### ♻️ Refactoring

- **helpers**: Remove duplicate import of _MAX_FLOAT64 in interest_rates.py


### ﻿fix

- **helpers,ir**: Replace struct-based NaN/Inf check with PyObject_RichCompare


### 👷 Build System

- Add typeCheckingMode to pyproject.toml for basic type checking


### 🧹 Miscellaneous

- Bump version to 0.0.12 [test release]


## [0.0.11-test] - 2026-06-08

### 🐛 Bug Fixes

- **helpers**: Struct-based scalar finiteness for -ffast-math wheels


### 👷 Build System

- Remove duplicate basedpyright section from pyproject.toml

- Fix invalid pyright executionEnvironments config


### 🧹 Miscellaneous

- Bump version to 0.0.11 [test release]


## [0.0.10-test] - 2026-06-08

### ✅ Testing

- **tests,build**: Fix TableBuilder symlink test and ignore N999


### 🎨 Styling

- **engine**: Normalize docstring LaTeX and formatting in helpers


### 🐛 Bug Fixes

- **ci**: Wait for Test PyPI index before wheel_test install

- **helpers**: IEEE754 finiteness under -ffast-math; strict batch identity tests

- **engine**: Batch-scalar identity under -ffast-math wheels

- **batch,helpers,ir,lt**: Round unit PV before benefits scaling


### 📚 Documentation

- **batch**: Document benefits batch-scalar identity contract


### 🧹 Miscellaneous

- **tests,build**: Fix test_45 imports and pyright test paths

- Bump version to 0.0.10 [test release]


## [0.0.9-test] - 2026-06-08

### ✅ Testing

- **tests**: Use tuple except syntax for Python 3.13 collection


### 🐛 Bug Fixes

- **ci**: Isolate no-license smoke test in wheel_test

- **ci**: Purge runner fingerprint across Keygen account in wheel_test

- **ci**: Run fingerprint cleanup script via bash shell

- **ci**: Serialize wheel_test jobs and fix fingerprint scope activation

- **tests**: Align Ubuntu wheel_test with Linux wheel 0.0.8 limits

- **tests**: Skip NaN finiteness cases on Linux wheel 0.0.8

- **tests**: Tolerate batch/scalar 1-ULP drift on Linux wheel 0.0.8 py3.12


### 📚 Documentation

- **ci**: Update Fase 9.B plan for wheel_test and cp312-cp314 matrix

- **ci**: Mark Windows cp312-cp314 wheel matrix complete

- **ci**: Record wheel_test GHA success and Linux wheel 0.0.8 compat (rev. 2.0)


### 🔧 CI/CD

- **release-test**: Add wheel_test matrix job with workflow_dispatch

- **release-test**: Add Keygen cleanup to wheel_test job


### 🧹 Miscellaneous

- Bump version to 0.0.9 [test release]


## [0.0.8-test] - 2026-06-07

### 🐛 Bug Fixes

- **tables,builder**: Wheel 0.0.7 unisex_blend init and scale parsing


### 🧹 Miscellaneous

- **skills**: Document LifeTable.__init__ and _parse_scale wheel patterns

- Bump version to 0.0.8 [test release]


## [0.0.7-test] - 2026-06-07

### 🐛 Bug Fixes

- **tables,helpers**: Wheel 0.0.6 delegate typing and inf finiteness


### 🧹 Miscellaneous

- **skills**: Document wheel 0.0.6 delegate and finiteness patterns

- Bump version to 0.0.7 [test release]


## [0.0.6-test] - 2026-06-07

### 🐛 Bug Fixes

- **tables,helpers,validators,growth_rates,utils**: Wheel batch and ffast-math validation

- **_activation,tests**: Cython-safe stdin and requests in activation

- **tables,tests**: Align wheel regex error assertions


### 👷 Build System

- **pyproject**: Include __main__.py in wheel


### 🧹 Miscellaneous

- **skills**: Document wheel 0.0.5 compatibility patterns

- Bump version to 0.0.6 [test release]


## [0.0.5-test] - 2026-06-07

### 🐛 Bug Fixes

- **validators,helpers,tables,_activation**: Prevent Cython annotation_typing coercion in wheels


### 👷 Build System

- **scripts**: Exclude _activation.pyi from wheel stub generation


### 📚 Documentation

- **api**: Align docstrings with Cython scalar coercion

- **scripts**: Document licensing stub exclusion and wheel audit patterns


### 🧹 Miscellaneous

- Document wheel scalar compatibility in lactuca-optimize skill

- Bump version to 0.0.5 [test release]


## [0.0.4-test] - 2026-06-07

### 🐛 Bug Fixes

- **api**: Coerce Python scalars for Cython wheel compatibility


### 🧹 Miscellaneous

- Bump version to 0.0.4 [test release]


## [0.0.3-test] - 2026-06-06

### Changelog

- Update changelog with new batch tests and documentation improvements


### ♻️ Refactoring

- Extract _scalar_or_arr_to_size, rename N->size, dedup isfinite (Cython STRICT + Ruff N806)

- **batch**: Remove 'in v1' from error messages; permanent limitation documented in PLAN

- **decrement**: Update version identifiers from V1/V2 to E1/E2 in comments

- **functional**: Fix duplicate Sequence import (collections.abc vs typing)

- **tests**: Rename test_batch_single_life → test_41_batch_single_life

- **tests**: Streamline array creation in batch tests for clarity

- **batch**: Remove unused mode variable and fix unicode escapes in joint batch helpers

- **tests**: Reorganize batch test suite and add coverage gaps

- Rename dict key total_bel -> total_pv for context-neutral API

- **batch**: Rename benefit= to benefits= in batch API

- **batch**: Redirect 2-life to n-life dispatchers; remove _*_batch_joint

- **engine**: Remove t_grid alias and redundant aliases from return_flows dicts

- **engine**: Rename t_payment -> payment_time in InsuranceEngine

- **engine**: Reorder return_flows dict keys in logical computation order

- **engine**: Remove redundant present_value from endowment dicts

- **engine**: Remove redundant float() wrappers in endowment arrays

- Rename benefit_* to benefits_* throughout life.py and functional.py

- **api**: Unify dict key payment_time/t_grid to time_grid in all engine dicts

- **batch**: Remove _axy_multi_table_dispatch and simplify 3 n-life dispatchers to single-call (13-A)

- **batch**: Route functional multi-table via OOP public API (13-B)

- **batch**: Inline single-life multi-table dispatchers into functional.py (14-A/B/C/D)

- **engine**: Remove dead-code n=1000 perpetuity guard in AnnuityEngine

- **engine**: Restore n: Union[float,None] signature in AnnuityEngine

- **tests**: Rename test_47/48/50 to test_45/46/47 (batch suite renumbering)

- **tests**: Remove duplicate scalar-batch identity tests D1+D2+D4, move D5 stress to test_41/42 (-22 test functions)

- **tests**: Reorder imports and simplify list comprehensions in test_45_batch_vector_params_multilife.py

- **tests**: Absorb test_45 into test_47 as S52-S55 (T-12A-01..62)

- **tests**: Update docstring for clarity on scalar/batch identity tests

- **tests**: Renumber test files 46-48 → 44-46 to reserve slots for new phases

- **ir**: Extract _annuity_batch_dispatch; eliminate ~240-line duplication between a() and ae()

- **batch**: Extract _aggregate_flow_parts helper to helpers.py

- **tables**: Recursive on_error='nan' in ax/ax/Ax; fix benefits rounding order

- Remove patch_benefits_validate.py script as part of benefits validation overhaul

- **tables**: Recursive on_error='nan' in joint/2-life/3-life annuity+insurance

- **tables**: Recursive on_error='nan' in nEx/nExy/nExyz/nEjoint

- Use public ax() API in functional single-table shortcut for per-policy m support

- **_activation**: Delegate _activate_interactive to _activate_with_key

- **tables**: Phase 6 - remove redundant import, docstring audit fixes

- **api**: Rename generate_payment_times to payment_times [**BREAKING**]

- Update execution checklist and mark steps as completed for renaming generate_payment_times to payment_times

- Extract DRY table-resolution and nan-result batch helpers (Fase 2 + Fase 3)

- Remove dead BatchErrorReport import from life.py

- **tables,engine**: Replace np.atleast_1d/asarray with _as_f64_safe


### ⚡ Performance

- Optimize engine/helpers.py (OOB guard, remove errstate overhead)

- Running log-sum in joint prob funcs, inline _joint_dbqx_func (no matrix alloc)

- Query _prob_decrement_interval once per life in _joint_survival_udd_yearwise_func

- Reuse age_defer and simplify error messages in _validate_actuarial_range

- NIVEL 2 R9 — replace np.add.at with np.bincount in return_flows scatter-add paths

- NIVEL 2 R10 — add R10 marker comment to continuous fallback loops in batch dispatchers

- PHASE 6.5 complete -- NIVEL 2-5 audit all pass, benchmark verified

- **engine**: OPT-A+B in AnnuityEngine/InsuranceEngine; fix test_50 S22/S23/S31

- Vectorise all 4 calculation modes for Ax and nEx batch dispatch

- **helpers**: Collapse redundant ndarray branches in batch detection and broadcast helpers

- **helpers,tables**: Eliminate redundant allocations in batch hot paths


### ✅ Testing

- **batch**: Reorganize n-life batch tests into test_38/test_39 dedicated files

- **batch**: PHASE 7.2 - functional multi-table batch tests (ax/ax-due/Ax/nEx)

- **batch**: PHASE 7.4 - functional single-table batch type contracts and OOP equivalence

- **batch**: Add full coverage for batch error handling and edge cases

- **batch**: Complete full coverage for all batch functions

- **batch**: Add select table batch coverage

- **batch**: Expand batch test suite with lowprio, single-life, 2-life, and select scenarios

- **batch**: Complete coverage gaps for nEx/nExy/äxy/Axy batch modes

- **batch**: Complete large-N and decimals coverage in test_44

- **decrement**: Add dedicated ResourceWarning threshold test

- **batch**: Add Section 24 per-policy InterestRate batch tests

- **batch**: Independent coverage tests Paso 1 (T-47..T-96)

- **batch**: Add T-45/T-46/T-59 t_output and simplified-mode tests for 2-life

- **batch**: T-44 axyz multitable return_flows and T-83 header updates

- **batch**: T-44b T-44c Axyz/nExyz multitable return_flows; fix docs redundancy

- T-17 — add test_benefits_on_error_nan_raises in test_40_batch_infra

- Add T-98..T-101 batch return_flows dot-product identity tests

- **batch**: Update 10 test files for unified "time_grid" dict key (13-C.4)

- Add batch-scalar exact identity test suite (test_50, S1-S17)

- Add multi-table exact identity tests (test_50 S18-S20)

- **test_50**: Add S26/S27 (mu_method+d/ts), S32 (cashflow_amounts×m)

- **test_50**: Add S47/S48/S49 — continuous n_frac, joint×m, fractional d

- **test_50**: Add S29/S35-S41 — perpetuity, frac-ts, piecewise-IR combos

- **batch**: Add S51 ajoint/ajoint all-modes scalar-batch identity (8 tests)

- **batch**: Add S56-S57 identity tests and test_48 P0/P1/P2 coverage for 3+ life methods

- **batch**: Replace rtol=1e-12 with assert_array_equal for scalar-batch identity tests

- **batch**: Fix 6 wrong-method bugs and add missing sections 14-18 in test_48

- Fill all P0/P1/P2 batch coverage gaps

- Add combined parameter broadcasting tests for 3-life/N-life (T12A71-78)

- Add fractional-n, t_output, and benefits coverage for 3-life/N-life batch (S24-S26)

- Cover R1-R5 batch gaps (äxyz simplified-raises, InterestRate 3-vida, scalar-float, dtype, stress)

- Cover P1-P3 batch gaps (force_integer_ts 3-vida, nExyz/nEjoint mutual exclusion, functional exact values)

- **ir**: PHASE 11 — add test_47_ir_batch.py (212 tests for InterestRate batch)

- **ir**: Add missing batch symmetry tests for ä(), on_error and cashflow_times exclusivity

- **ir**: Add gap-coverage tests for all-invalid batch, perpetuity+deferment, degenerate return_flows

- **batch**: Add missing benefits rf_false/on_error_nan/het_m tests; fix batch_calculations.md residuals

- **batch**: Update batch tests for relaxed benefits parameter contract

- **tables**: Regression tests for batch nan recursive paths + rounding fix

- **ir**: Extend test_47_ir_batch coverage for on_error='nan' edge cases

- Add missing cashflow_times/cashflow_amounts tests for Axy/Axyz/Afirst and due-annuity methods

- Add exhaustive cashflow constraint tests for annuity functions and fix äxy cashflow_times validation

- Complete 100% cashflow coverage - m>1 multi-life batch==scalar and validation error tests

- Complete §N3 record_ids coverage to 12/12 multi-group functional functions

- Add S72 ValueError guard tests for t_output without return_flows

- **cli**: Add helper contract coverage for license commands

- **cli**: Cover edge cases for license helper contract

- **tables**: Add cartesian+return_dict+ir_sequence alignment test (coverage gap)

- **tables**: Phase 6 - smoke tests for cartesian/dict/blend/warnings

- **tables**: Rename smoke tests to follow test_NN convention

- **activation**: Update tests for Fix 1/Fix 2; protect license.json in conftest

- **batch**: _arraylike_len_or_1 unit tests; _detect_batch_joint Series regression; t_output Series NDArray invariant

- **batch**: Complete Series audit coverage and module pd/pl imports

- **ir,gr**: Add regression tests for deepcopy and copy() methods

- **batch**: Add T-46 test for uniform benefits scaling


### ✨ Features

- Add PHASE 0 batch helpers and validators

- AnnuityEngine.discrete_precision_batch - phase 1.1 vectorized batch engine

- InsuranceEngine.discrete_precision_batch - phase 1.2 vectorized batch engine

- EndowmentEngine.discrete_precision_batch - phase 1.3 vectorized batch engine

- _annuity_batch dispatcher in life.py - phase 2.1 vectorized batch dispatcher

- _insurance_batch dispatcher in life.py - phase 2.2 vectorized batch dispatcher

- _endowment_batch dispatcher in life.py - phase 2.3 vectorized batch dispatcher

- Batch mode for ax, äx, Ax, nEx public methods - phase 3.1-3.4

- Multi-table batch dispatchers in life.py - phase 4.1-4.4

- Joint-life batch helpers and batch mode for axy/axy/Axy/nExy - phase 5

- **functional**: Add multi-table batch dispatch to functional API (PHASE 6)

- **batch**: Add n-life batch dispatchers and functional API batch support

- **batch**: Add per-policy vector_gr (VG-1) and vector_m (VM-1) support

- **batch**: PHASE 6.6.A — _check_batch_param limit 50, _validate_actuarial_range_batch collect-all

- **batch**: PHASE 6.6.A — collect-all validation in all 16 batch functions

- **batch**: PHASE 6.6.B - on_error/record_ids in all 16 batch functions

- **batch**: PHASE 7.6 - vector_gr and vector_m for n-life batch (VG-1/VM-1)

- **batch**: Add _check_batch_ts_integer and _fmt_batch_violation helpers

- Export BatchResult and BatchErrorReport; document on_error batch handling

- **config**: Replace config alias with dynamic _ConfigProxy for singleton safety

- **batch**: Extract _aggregate_flow_parts; enable return_flows in n-life batch

- **batch**: Add benefit= per-policy monetary weighting to single-life batch

- **batch**: Benefit= per-policy monetary weighting in n-life batch

- **batch**: Enable return_flows in continuous_precision mode

- Enable return_flows=True and t_output in multi-table functional joint dispatch

- **batch**: Complete PHASE 10 and update checklists in PLAN_vectorizacion_batch.md

- **batch**: Vectorise ts, d, n, m, ir in all multi-life batch methods (Fase 12-A)

- **batch**: Per-policy mortality tables for all multi-life batch methods (Fase 12-B)

- **tables**: Normalize n=np.inf sentinel in Axyz joint method (S50)

- **ir**: PHASE 11 — batch route for InterestRate.a() and InterestRate.ä()

- Add benefits parameter to InterestRate.a() and ä() for return_flows=True

- **ir**: Extend InterestRate.a() and ä() with benefits parameter for return_flows=True

- **annuity**: Add simplified core computation for annuity calculations to optimize performance

- **insurance**: Implement single-pass core for discrete_simplified in InsuranceEngine and add regression tests

- **annuity**: Simplify AnnuityEngine code and improve type hints; enhance test coverage for InterestRate broadcasting

- **batch**: Allow benefits parameter without return_flows=True in batch annuities

- **lt**: Replace 16x benefits blocks with _validate_benefits helper

- Add cashflow_amounts to annuity-due methods (äx, äxy, äxyz, äjoint)

- Add fail-fast ValueError guard for t_output without return_flows=True

- Add on_error scalar guard to all 16 batch methods

- Implement on_error='nan' scalar guard for batch methods

- **api**: Guard record_ids in scalar mode for all 16 batch OOP methods (Bug F) [**BREAKING**]

- **cli**: Implement license refresh status doctor commands

- **tables**: Phase 1 — add TableKey namedtuple, export from __init__, 17 tests

- **tables**: Phase 2 — table_name Sequence, range/None cohort/duration, unisex_blend Sequence, 30 tests

- **tables**: Phase 3 - cartesian=True, pre-validation, combos abstraction, 17 tests

- **tables**: Phase 4 - return_dict=True, TableKey dict keys, 15 tests + E2E

- Batch multi-life methods triggered by non-ages scalar params

- Accept Pandas/Polars Series directly as batch input parameters

- Add .cursorrules file for Lactuca Actuarial Library guidelines and standards


### ﻿chore

- **scripts**: Close G-13 in lactuca-series-batch skill audit matrix


### ﻿docs

- **batch,functional,ir**: Update batch guide and cross-ref pages for ir/gr Series

- **batch**: Remove redundant local TOC directive


### ﻿fix

- **helpers,batch**: Preserve ir/gr objects in object-dtype Pandas/Polars Series


### ﻿test

- **batch,tests**: Add S41 Series object-dtype ir/gr piecewise coverage


### 🎨 Styling

- Apply Ruff formatting to annuity.py


### 🐛 Bug Fixes

- Update project description and homepage URL in pyproject.toml

- Update license section in README.md for clarity and add activation instructions

- Update implementation plan with completion dates for steps 8.1 and 8.2

- Cython STRICT fixes in engine/helpers.py (M-3 int cast, M-4 _prob_life_interval)

- Guard log(0) and replace np.isfinite with IEEE-safe check in MuApproximator (-ffast-math)

- Consistent 1e-300 guard and inline in _individual_avg_forces (-ffast-math)

- Rename N_k->n_k, M->n_t in discrete_precision_batch (Ruff N806)

- Corregir bugs y optimizar dispatch multi-tabla en functional.py

- **life**: Update error messages for return_flows in batch mode to clarify calculation_mode requirements

- **functional**: Add error handling for return_flows in joint-life multi-table batch for axy and äxy functions

- **batch**: Resolve ir=None to table default in batch helpers; add PHASE 7.7 tests

- **life**: Accept n=+inf as whole-life sentinel in ax/ax-due/Ax scalar paths

- **tests**: Update vectorial cohort tests for ResourceWarning threshold

- **tables/life**: Correct ir type hints and docstrings to include list[InterestRate]

- **batch**: Emit UserWarning for fractional ts when GrowthRate active

- **config**: Reset_to_defaults() preserves tables_path

- **config**: Resolve Config forward reference in __deepcopy__ return annotation

- **batch**: Pre-fase-a production bugs (aliasing, exception types, dead code, silent discard)

- **batch**: Endowment return_flows 2D shape, ravel defense, T-58 continuous_precision

- Replace NotImplementedError with ValueError for vector-m + return_flows

- **engine**: Fix 3 batch precision bugs to enable exact scalar identity

- **batch**: Perpetuity per-policy fallback in _annuity_batch for zero-tolerance

- Replace np.asarray with _as_f64_safe in public API boundaries

- **helpers**: Correct BatchErrorReport docstring reference from pandas to polars

- **ir**: Fix stale ä() Notes docstring; fix stale on_error=nan inline comments; add missing scalar-batch identity tests (G1/G2/G3)

- **ir**: Correct return types, remove broadcast copies, align valid_idx, fix docstring

- Reorder cashflow params in insurance signatures to times-first

- Propagate on_error/record_ids in multi-table functional single-group shortcut

- Propagate on_error/record_ids in multi-table functional multi-group loop

- **api**: Add _validate_on_error fail-fast and record_ids length guard to all 12 joint/n-life functional wrappers (Bug I)

- **api**: Guard joint-life OOP methods against empty/invalid ages type; fix stale test_40

- **api**: Guard joint-life OOP methods against empty tables_yz/tables_others

- **api**: Accept np.ndarray as ages in all 11 joint/n-life OOP guards

- **api**: Raise ValueError for batch-only params used in scalar mode

- **_activation**: Convert LactucaLicenseError subclasses to clean SystemExit in __init__

- **_activation**: Recover from LAC-3001/3002/3003/3004 using stored key; never delete license.json

- **_activation**: Online revalidation for expired/invalid-fp; add python -m lactuca activate CLI

- **tests**: Remove unused exc_info in test_activate_calls_activate_interactive

- **_activation**: Replace isatty guard with _was_ever_activated(); show [T/K/Q] in Jupyter

- **_activation**: Block trial requests on paid devices

- **cli**: Harden license refresh and align server states

- **license**: Remove user env dependency from CLI diagnostics

- Plan — gap analysis 7: unisex_blend_list cartesian extend, range cohort, _resolve helper, E2E test description

- **_activation**: Atomic write for license.json via tmp+os.replace

- Batch mode triggered by m/gr arrays in ax/äx/Ax/ir.a/ir.ä and ir in nEx

- Update batch detection to include `m` and `gr` in annuities

- Normalize_n_sentinel, remove cashflow_times from äxy OOP signature, and add validation guards in joint-life methods

- Update sphinx-build status to indicate successful build and zero warnings

- Homogenize ir annotation in nExyz and nEjoint functional signatures (GAP-39)

- **batch**: Rename 'policies' to 'records' in error messages and docstrings

- **activation**: Redirect non-keygen expired keys to interactive activation

- Accept Pandas/Polars Series for batch parameter m

- Accept Pandas/Polars Series for batch parameters ir and gr

- Normalize record_ids to list for safe positional indexing with any sequence

- **life**: Normalize t_output via _as_f64_safe in all batch methods

- **helpers**: Accept pd/pl Series in _validate_numeric_input allow_sequence path

- **dates**: Accept Pandas/Polars Series in is_leap_year, days_in_year, days_in_month

- **tests**: Restore _is_integer_valued import in integer detection helpers

- **ir,gr**: Add InterestRate.__deepcopy__ and copy() to IR and GR

- **notebooks**: Update error handling in pycactus_legacy notebook

- **workflows**: Improve error handling in validate-wheels.yml for license checks


### 📚 Documentation

- **plan**: Reconcile Regla 3 in batch engine steps 1.1-2.3 and add Cython STRICT speed principles

- **plan**: Add implementation phase guidelines for batch vectorization process

- Mark PHASE 1.1 as completed in batch vectorization plan

- Mark PHASE 1.2 as completed in batch vectorization plan

- Mark PHASE 1.3 as completed in batch vectorization plan

- Mark PHASE 2.1 as completed in batch vectorization plan

- Mark PHASE 2.2 as completed in batch vectorization plan

- Mark PHASE 2.3 as completed in batch vectorization plan

- Mark PHASE 3.1-3.4 as completed in batch vectorization plan

- Mark PHASE 4.1-4.4 as completed in batch vectorization plan

- Mark PHASE 5.1-5.3 as completed in batch vectorization plan

- **plan**: Document VG-1 vector_gr and VM-1 vector_m vectorization gaps

- **plan**: Mark PHASE 6.4 complete, update PHASE 7 status with test_38/test_39

- **plan**: Mark VG-1 and VM-1 as COMPLETADO, update Sequence type hints

- **plan**: Add PHASE 6.6 batch error handling plan and PHASE 7.7 tests

- **plan**: Add D-1 gap — generational/select table cohort-duration batch limitation

- **plan**: Mark PHASE 6.6.B as completed (all 16 functions)

- **plan**: Document m heterogeneous + return_flows=True permanent limitation

- **plan**: Mark PHASE 7.7 completed; document ir=None batch bug fix

- Update PLAN_vectorizacion_batch.md - mark 7.1/7.2/7.4/7.5/7.6 completed

- PHASE 8 complete — batch mode docstrings for all public methods in life.py and functional.py

- Fix arrow notation consistency in docstrings

- PHASE 9 — batch calculations user guide and cross-references

- Fix batch API examples and add cross-references to batch_calculations

- Add batch_calculations cross-references to cookbook, using_tables, prospective_reserve

- Remove vectorial hard limit; add memory warning + group-then-update pattern

- **plan**: Mark 6.6.B.1, 9.2-9.6 as completed; verify 9.5/9.6 autodoc

- Update references in licensing FAQ for consistency and clarity

- Update changelog with refactoring, performance improvements, testing enhancements, and new features

- **9.7**: Add probable_flows.md — IFRS 17 BEL, SCR SII, IAS 19 PVDBO, rate/mortality sensitivity

- Correct m parameter description — supports per-policy sequences in batch mode

- Document cashflow_times per-policy limitation and group-by-product workaround

- **plan**: Mark implementation complete — all phases done, PR pending

- **plan**: Add pre-merge checklist with 3 validation items

- **batch**: Add broadcasting rules table and per-policy InterestRate examples

- **probable-flows**: Fix cashflow_amounts usage for per-policy sums insured

- **changelog**: Add decrement ResourceWarning threshold entries

- **batch**: Clarify per-policy intro — name cashflow exceptions inline

- **batch**: Make per-policy examples self-contained; remove numpy dependency

- **batch**: Add print() output to all runnable code snippets; fix cohort table name

- **audit**: Add comprehensive documentation audit protocol for verifying and updating markdown files

- **batch**: Correct formatting in output table for clarity

- **batch**: Use ax (postpagable) for benefit/pension examples; äx only for premium formula

- **batch**: Fix table selection per Spanish regulation (PER2020 ahorro/pensiones, PASEM riesgo); recalculate all output values with cohort=1960

- **user-guide**: Update batch_calculations.md with benefit= examples and flow aggregation patterns

- **plan**: Add Phase 10 benefit= batch plan with 61-gap analysis and 44 tests

- **plan**: Add G-62..G-64 (t_output gaps, T-43 numeric assertions, T-45/T-46)

- **plan**: Add G-65..G-68 (t_output äx/nEx OOP, t_output functional, on_error/record_ids n-vida, record_ids f_Axy/f_nExy) and T-47..T-54

- **plan**: Add lactuca-optimize principles section with Cython STRICT checklist for src/ implementation

- **plan**: Add Bloque 4 (docstrings actuarial-docstring skill) and Bloque 5 (docs user guide lactuca-docs) to implementation principles

- **plan**: Add G-69..G-75 (CI regression, aliasing, simplified-mode guards, nExy coverage) and T-55..T-65 to implementation plan

- **plan**: Add G-76..G-81 (production aliasing bug, error-type mismatch, dead code, Ax/nEx mode-raises gap, is-not assertion gap, on_error+return_flows gap) and T-66..T-70

- **plan**: Add G-82..G-86 (Axy/nExy multi-table silent return_flows, nEx warning gap, force_integer_ts 2vida gaps, test_39 functional NotImplementedError) and T-71..T-75

- **plan**: Correct T-71/T-74 fix strategy (guard not forward), mark T-64 redundant, add G-87..G-89 and T-76..T-78

- **plan**: Restore G-86..G-89, add G-90..G-94 (test_38 all-invalid/empty-batch/record_ids/continuous_simplified gaps; test_42 matrix housekeeping) and T-79..T-83

- **plan**: Add G-95..G-101 and T-84..T-90 (secondary-life invalid, fractional n, due annuity gr/m, continuous_simplified functional, single_group batch, functional gr, generational tables gaps)

- **plan**: Add G-102..G-107 and T-91..T-96 (select+SelectGen batch coverage gaps: return_flows, t_output, SelectGen, äjoint/äxyz, functional API, on_error)

- **plan**: Add G-108 and T-97 (return_flows+benefit batch aggregation vs scalar loop)

- **plan**: Add ordered implementation roadmap (Pasos 0-7) with conflict-free commit sequence

- **plan**: Fix ordering issues in Pasos 2-6 (aggregate_flow_parts dep, A-1.16 dupe, G-82/G-85 confusion, B-5 ordering)

- **plan**: Assign T-97 to Paso 3 (benefit+return_flows batch vs scalar loop cross-check)

- **plan**: Assign T-75 to Paso 2 (covered by T-56); T-64 correctly unassigned (superseded)

- **plan**: Update PLAN_batch_flujos_benefit status to Paso 4 completed

- **batch**: Add benefits= and return_flows docstrings and user guide updates

- **plan**: Record Paso 8 bug fixes in PLAN_batch_flujos_benefit

- **plan**: Update status to Pasos 0-8 completados; add ac46f9b commit entry

- **plan**: Registrar T-44 como completado y añadir commit 2dbcfd1 al historial

- Doc-16 — clarify t_grid vs payment_time two-level key design

- Update plan checklist — mark 51 T-xx items as done (stale since Pasos 0-8)

- Finalize plan — add Doc-16/checklist commits to history, clear pending section

- Mark all 5 doc-plan items as done in Plan de documentacion section

- Mark Fases A-B-C-D checklist items as done (stale since Pasos 0-8)

- Fix Approach B EIOPA snippet and note blocks in batch_calculations.md

- Add parameter compatibility table and t_output/return_flows limitation note

- Update compatibility table and Fase 11 plan after Tarea D

- Remove stale NotImplementedError entries from axy/äxy/ajoint/äjoint docstrings

- Update changelog with recent batch and engine API changes

- Add Fase 12 planning for multi-life vectorized parameters

- **batch**: Per-policy mortality tables for multi-life batch methods (Fase 12-B/C)

- Register Fase 13-C completion in PLAN_batch_flujos_benefit

- **batch**: Rename t_grid/payment_time to time_grid in docs and functional.py docstrings (13-C.5)

- Update plan -- mark 13-C.5 and 13-A as completed

- Update plan -- mark 13-B as completed (ec86775)

- Update plan -- mark Fase 14 as completed (6537a73)

- Update plan -- mark Fase 15 (batch-scalar exact identity) as completed

- Update PLAN_batch_flujos_benefit -- mark Fase 15 as completed

- Update PLAN_batch_flujos_benefit -- add Fases 13-15 to status table

- Update Fase 15 tracking -- mark bug fixes + identity tests as completed, defer DGEMV to future

- Document DGEMV cancellation rationale (FP non-assoc) + safe OPT-A..E alternatives

- Fase 16 tracking corrected + S21-S30 test gaps documented

- Fase 15 tracking corrected + GAP-NEW-K..V documented; remove GAP-T50-J (already covered)

- Update plan -- mark S50 (n=np.inf joint normalization) as completed (0d31036)

- Add Fase 16 pre-merge checklist and release sequence to plan

- Update plan with S51 (606 tests, commit af64d1c)

- Improve documentation clarity and consistency in annuity.py

- Clarify return_flows scalar vs batch dict schema

- **functional**: Complete return_flows docstring in ax, äx, axyz, äxyz, Ax, Axyz, nEx

- **life**: Fix batch return dict key t_grid → time_grid in docstrings

- **tables**: Update return_flows docstring — supported also for continuous_precision

- **ir**: PHASE 11 step 11.4 - update a() and ae() docstrings for batch mode

- **user_guide**: PHASE 11 step 11.6 - add Pure financial annuities section to batch_calculations.md

- **plan**: Mark Phase 11 steps 11.4-11.7 complete

- **lactuca**: Fix docstring audit P1-P3 corrections

- Update Python version requirement to 3.12 and add notes on 'm=14' scheme in user guides; enhance DecrementTable with stable identity hash and equality checks

- Audit and fix batch_calculations.md and interest_rates_guide.md

- Fix simplified mode -> simplified modes (plural) in compat table

- **ir**: Fix stale on_error=nan+return_flows Notes in a()/ä() docstrings and batch_calculations.md; mark IMPL plan 100% complete

- Update changelog and implementation notes for batch NaN+benefits

- **skill**: Expand lactuca-optimize skill with memory, Cython, and precision patterns

- Document cashflow_amounts in due-annuity methods and clarify cashflow_times design

- Document calculation_mode restriction in insurance cashflow_times and add insurance section to irregular cashflows guide

- Mark INC-4 and INC-5 as COMPLETADO in impl plan

- Update batch `on_error='nan'` DRY optimization plan with detailed objectives and validation updates

- Fix cashflow_times/cashflow_amounts parameter order in insurance docstrings

- Document on_error/record_ids multi-table support in docstrings and user guide

- Add on_error='nan' example for multi-table functional joint/n-life dispatch

- Add IMPL plan for missing vectorised batch engines (Ax/nEx)

- Document t_output guard (ValueError) across docstrings and user guide

- **plan**: Add implementation plan for on_error/record_ids multi-table forwarding fix

- **helpers**: Fix _round_to_nearest_int docstring examples to show near-integer use case

- **functional**: Complete on_error/record_ids Raises + examples in ax, äx, Ax, nEx

- **errors_reference**: Update LAC-3002/3003/3004 recovery sections for auto-recovery

- **batch**: Update batch_calculations user guide for t_output/on_error guards

- **plan**: Mark all checklist items completed in IMPL_activation_recovery_fix

- **impl**: Mark §11 paso 6 manual verification complete

- **licensing**: Align internal config docs

- **licensing**: Finalize S-3 documentation

- **plan**: Finalize license CLI implementation plan

- Update changelog and keygen license notes

- **licensing**: Finalize CLI docs and validation checklist

- **licensing**: Mark implementation plan as fully completed

- **user-guide**: Align batch flow contracts and examples

- **changelog**: Update documentation entries

- **user-guide**: Remove redundant notes in batch calculations

- Fix t_output docstrings — clarify bucketing mechanism and total_pv invariant

- **batch**: Improve portfolio examples, simplify constructor usage, fix sexes array

- Plan — document cartesian-vs-groupby distinction; portfolio canonical pattern in Phase 5

- Plan — gap analysis 6: existing-doc update coverage (notation_glossary, tables_taxonomy, disability/exit API, decrement unconditional, using_tables param updates, batch forward-ref)

- **tables**: Phase 5 - TableKey, cartesian, return_dict, unisex_blend docs

- Mark all plan checkboxes complete in PLAN_table_cartesian_dict.md

- **batch**: Add zip-mode comments to all multi-sex LifeTable examples

- **batch**: Simplify per-policy table examples with vectorial constructor pattern

- **changelog**: Update auto-generated changelog entries

- Replace np.arange with payment_times in code snippets

- Replace unnecessary np.array with plain lists in code snippets

- Mark FEAT_batch_multilife_nonages plan as 100% COMPLETADO

- Audit and fix batch_calculations.md

- Fix missing MyST cross-reference anchors in batch_calculations.md

- Move Error handling section before Single-table batch for pedagogical clarity

- Add BatchResult and BatchErrorReport to API reference and fix cross-references

- **changelog**: Add entries for batch DRY helpers and docs updates

- **notebooks**: Update pycactus_legacy with batch on_error example

- Fix multi-line message indentation in on_error example

- Fix sphinx warnings - duplicate BatchResult attrs, np.bool_ RST ref, batcherrorreport anchor

- Add premium column to Polars integration example

- Document Pandas/Polars Series acceptance for all batch params

- Document Pandas/Polars Series acceptance for benefits in all batch methods

- **batch**: Document Series acceptance for n/d/ts and ages in all multi-life methods; fix DRY in IR batch dispatch; add 30 Series coverage tests

- **functional,interest_rates**: Add Pandas/Polars Series mentions to all per-policy param docstrings

- **functional**: Add Pandas/Polars Series mentions to multi-life param docstrings

- **life**: Add Pandas/Polars Series mentions to OOP method param docstrings

- Document Pandas/Polars Series support in batch API user guide

- **impl**: Extend Series audit with DA-16..DA-22 docstring gaps and CB-01..CB-03 bugs

- Add guidelines for Conventional Commits and versioning workflow

- **functional**: Fix docstring indentation for gr parameter in ax, ax, axy

- **batch,docs**: Audit batch_calculations and probable_flows guides

- **ir,gr**: Document copy() snapshotting in IR and GR user guides

- **docs**: Remove redundant float() wrapping on NumPy results in probable_flows

- **user_guide**: Enhance cashflows documentation with batch mode example

- **user_guide**: Update probable_flows documentation for annuity calculations

- **plan**: Update batch plan for direct merge to main

- **plan**: Close batch vectorization plan after merge to main

- **plan**: Align CI/CD plan with Gate v0.1.0 rev 1.7

- **changelog**: Add missing entries from batch, activation and Series work


### 🔧 CI/CD

- **docs**: Fix gh-pages deploy with CNAME and rsync multi-version


### 🧹 Miscellaneous

- Update actuarial data table and legacy notebook

- Mark Fase 12-B/C/D/final as completed in PLAN tracking table

- **lint**: Add N806 to functional.py ruff ignores

- **copilot**: Improve docstring-auditor and docs-writer agents

- **copilot**: Add engine instructions, new-test prompt, and batch/profile skills

- **dev**: Add git hooks (commit-msg, pre-commit, pre-push) and installer

- **notebook**: Update pycactus legacy notebook

- Add cartesian+dict LifeTable constructor implementation plan

- Update TODO — mark vectorized functions and licensing as done, add docs audit template

- **notebook**: Update pycactus legacy notebook

- Complete plan coverage audit — 10 missing test paths + explicit out-of-scope section

- Plan gap analysis — 12 missing tests + cross-phase fixes + 3 new risks

- Plan deep-gap analysis — fix phase-order bugs, restore 3 lost tests, rename dedup test, add 4 new tests

- Plan docs gaps — add actuarial-docstring skill (ph1-4), Docs-writer agent (ph5), Docstring-auditor agent (ph6), restore ph3 header

- Plan gap analysis 4 — 7 impl/test gaps fixed (cartesian guard, combos refactor, Phase4 thread clarity, API docs, 3 missing tests)

- Plan — document tables[0] limitation, memory footprint tradeoff, add lazy-construction and int-subscript to out-of-scope

- Plan — gap analysis 5: fix test class placement, unisex_blend_list always-produce note, api/base.md step, KeyError smoke test

- **notebooks**: Update pycactus_legacy with per-policy table pattern example

- Fix test module docstring and mark rename plan as completed

- Migrate AI instructions to Cursor rules and skills

- Add lactuca-series-batch skill and agent index

- **skills**: Add actuarial and activation audit workflows for Fase 9

- Bump version to 0.0.3 [test release]


## [0.0.2-test] - 2026-04-30

### ♻️ Refactoring

- Streamline E2E license test scenarios and improve Python script execution


### ✅ Testing

- **activation**: Complete test suite for _activation module (114 tests)

- **e2e**: Add G.NEW-A and G.NEW-B manual scenarios for FASE N MAC


### ✨ Features

- **webhook**: Add name field to trial endpoint (B.4c)

- **licensing**: Add license verification and activation module

- **pricing**: Add initial pricing policy document

- Add E2E manual testing workflow for license system across platforms

- Add tests for LACTUCA_CONFIG_DIR license.json handling in verify_or_activate()

- Update Phase G action plan with new scenarios and workflow enhancements

- **licensing**: Remove annual billing variants and restructure Keygen policies

- **tests**: Add manual test helper script for Fase G offline/revocation scenarios

- **docs**: Add EULA acceptance checkbox gate in pricing.md; update F.6 IMPL with implemented solution

- **docs**: Replace checkbox banner with modal gate for EULA acceptance (option C)

- **webhook**: Bilingual purchase email with plan name, security hardening, null-safe JSON access

- **activation**: Add FASE N — MAC integrity layer (LAC-3003/3004/3005)


### 🐛 Bug Fixes

- **ci**: Drop cp311, add CMAKE_BINARY_DIR_OVERRIDE for Windows, fix Python 2 except syntax

- **tables**: Parenthesize multiple exception types in builder.py

- **webhook**: Sanitize user inputs to prevent HTML injection and URL path injection

- **webhook**: Harden against empty secret, key HTML injection, and license ID path injection

- Handle NO_MACHINES on first activation by registering machine then re-validating

- Surface Keygen HTTP error in CI logs and guard against data:null in NO_MACHINES response

- Remove dead try block in _create_machine; guard metadata:null; document Keygen policy auth requirement

- Update dashboard configuration from API Tokens to License Keys for machine registration

- Harden license activation against Keygen null responses and Python 2 syntax

- Add diagnostic error codes [LAC-XXXX] and context to all license errors

- Pre-commit audit _activation.py — 3 issues (DEAD + 2 IMPORTANT)

- Set real Ed25519 verify key in _KEYGEN_VERIFY_KEY

- Handle incompatible license schema version in local verification

- **tests**: Mock _get_hardware_fingerprint in signature-check tests

- **ux**: Fix 5 activation UX bugs in _activation.py

- **ux**: Add feedback during periodic license revalidation

- **ux**: Replace unicode escapes with literal chars; confirm revalidation

- **ux**: Suppress traceback after ActivationRequiredError in __init__

- **ux**: Eliminate message duplication and Windows UnicodeEncodeError

- **activation**: Add nonecheck guards on list[0] access; clean import aliases

- **activation**: Distinguish unsigned license from tampered (LAC-3001)  - Add _STATUS_UNSIGNED constant for Keygen policies without Ed25519 signing - _verify_local(): keys absent (None) → LAC-3001; both empty strings → _STATUS_UNSIGNED - _do_verify_or_activate(): force needs_revalidation=True when _STATUS_UNSIGNED - scripts/setup_manual_tests.py: fix platformdirs path (appauthor=False) - tests: add 3 new tests for _STATUS_UNSIGNED behavior (119 total, all passing)

- **activation**: Add trailing newline to error messages shown via SystemExit

- **e2e**: Broaden G.1/G.3 success check to match 'license' in error output

- **docs**: Correct LS checkout URLs; add Spanish EULA link in checkbox

- **webhook**: Send purchase email with license key after order_created event

- **docs**: Update checkout URLs in pricing section to new store domain

- **webhook**: Close EULA modal on checkout, Spanish email for ES buyers, rename purchase sender to licenses@

- **webhook**: Add lang detection logging, add country name fallback for Spanish detection

- **webhook**: Default to Spanish in LS Test Mode (billing_address is null in test)

- **docs**: Migrate all URLs from actuaan.github.io/lactuca to lactuca.actuaan.com; centralize via myst_substitutions and _DOCS_BASE_URL

- **_activation**: Migrate URLs, add lang detection and email confirmation in trial flow

- **_activation**: Add trailing newline to all user-facing messages

- **webhook**: Remove hardcoded device limit from trial emails

- **_activation**: Refresh tier from Keygen on every revalidation

- **ux**: Add trailing newline to stderr messages and fix G8 setup script

- Add missing newlines for better readability in stderr messages

- Update trial email legal text and clarify license suspension process

- **docs**: Update implementation plan with new phases and improve changelog headings

- **docs**: Update comments for license scenarios and clarify license requirements

- Optimize license verification by caching verify key bytes and improving error handling

- **tests**: Enhance license verification tests by patching key bytes and improving error handling

- **tests**: Add ignore rule for extraneous parentheses in exception handling

- **config**: Migrate domain to lactuca.io and update support email

- Update Resend email addresses to use lactuca.io domain

- **resend**: Use resend v2 API (emails.send lowercase)

- **resend**: Revert to Emails.send() uppercase (SDK v2 API)

- Correct README content, domain to lactuca.io, and license contact email

- Correct cibuildwheel Windows path escaping causing LNK1104 linker error

- Align release-test Windows build config with working validate-wheels workflow

- Redirect ARCHIVE_OUTPUT_DIRECTORY per-target to prevent LNK1104 on Windows MSVC


### 👷 Build System

- Drop Python 3.10/3.11 support, min version 3.12


### 📚 Documentation

- **impl**: Update security audit §10.7 with Vercel Firewall decisions and Upstash deferral (rev 21)

- Add pricing, activation, faq_licensing, and EULA pages (Fase F, F.1-F.5)

- Add licensing pages (EULA, pricing, activation, FAQ, privacy) and fix sphinx build

- Update EULA with detailed licensor information, license terms, and trial conditions

- **licensing**: Add licensing, pricing, EULA and FAQ documentation

- **internal**: Update distribution licensing implementation notes

- **legal**: Simplify prompt transcripts, remove fragile error subtexts, sync heartbeat wording

- **legal**: Fix trial registration channel in privacy policy (prompt, not web form)

- **legal**: GDPR compliance gaps in privacy policy (Art. 13, 22, LSSI NIF)

- **legal**: GDPR Art. 13(2)(e) mandatory-data disclosure, IP in activation, Art. 22(2)(a), opt-out link

- **legal**: Add postal address (LSSI-CE Art. 10), correct fingerprint pseudonymity (GDPR Art. 4(5))

- **legal**: Add IP to trial registration, stored-where to §2.4, fix retention label (LGT/contract)

- **legal**: Disclose Vercel access log retention in §4 (GDPR Art. 13(2)(a))

- **legal**: Remove file-format enumeration from §5.3 to avoid contractual staleness

- **legal**: Fix §3 anchor name, fix FAQ cross-ref to §4, add last-updated date to EULA

- **legal**: Add OEM misuse to §8.2(b) no-refund exceptions to match §16.1 claim

- **legal**: Fix 3 discrepancies in faq_licensing (device limits, heartbeat grace behavior, Keygen server location)

- **legal**: Fix trial expiry wording, offline warning text, academic eligibility, Enterprise named-user count in FAQ

- **legal**: Fix heading levels for platform subsections, fix misleading Keygen portal link in activation guide

- **legal**: Update Windows license path and clarify license validation grace period details

- **legal**: Add LicenseSeatExhaustedError to troubleshooting, add explicit anchor in FAQ

- **i18n**: Add complete Spanish translations of EULA and Privacy Policy; update product tagline to Life Actuarial Calculation Library

- **planning**: Add web design implementation plan for Sphinx docs

- Update contact email to support-lactuca@actuaan.com in README and support documentation

- **pricing**: Update pricing tiers and add Founding Members Early Bird offers

- **changelog**: Update changelog with new features, bug fixes, and legal compliance updates

- Add new documentation files and update licensing information

- Add dead code review step to lactuca-optimize skill

- Update pricing to definitive prices; team/enterprise annual via direct invoice

- **impl**: Mark Fase G manual tests G.4/G.5/G.5b/G.8 as passed (2026-04-24)

- **impl**: Mark Fase G workflow scenarios G.1-G.9b as passed; G.10 pending

- **impl**: Activate LS store; unblock F.6/F.7/A.5b/K.3; add F.8 replication task; replace Fase H with CI/CD Fase 10 reference

- **impl**: F.6 rework — no native LS checkbox; document precontractual description strategy with legal basis

- **impl**: F.7 rework — clarify LS receipt vs Resend key email; add confirmation modal and thank you note texts

- Add FASE N verification flow and LAC-3003/3004/3005 to public docs

- **impl**: Fix verification barrier order; update IMPL and CI/CD plan

- **impl**: Mark Fase G fully completed (G.NEW-A/B PASADO 2026-04-28)

- Update CI/CD plan (version accumulation fix, sync-public trigger) and project notes


### 🔧 CI/CD

- Add LACTUCA_LICENSE_KEY to build verification step in e2e workflow

- Fix cross-platform pip install line continuation (Windows PowerShell)

- Expose KEYGEN_ACCOUNT_ID at job level for all e2e steps and subprocesses

- Remove post-install import check from build step (activation tested in G.9)

- Add CI teardown step to deactivate machine from Keygen after E2E run

- Simplify machine cleanup to use License-key auth, delete all machines

- **e2e**: Include Python version in pip cache key to prevent cross-version cache pollution

- Rewrite release.sh with test/prod/bump modes; automate sync-public after prod release; add validate-docs header


### 🧹 Miscellaneous

- **lint**: Suppress UP045 (Optional[X] vs X|None) globally — Cython incompatible

- Update pricing policy spreadsheet

- Update pricing policy spreadsheet

- Update changelog with new features, bug fixes, and CI/CD enhancements

- Bump version to 0.0.2 [test release]


## [0.0.1-test] - 2026-04-17

### ♻️ Refactoring

- Remove unused continuous_simplified_B method from AnnuityCore class

- Remove unused _resolve_interest_rate function from core.py

- Improve error handling in _validate_numeric_input for finite values

- Update return type annotations and improve validation logic in LifeTable methods

- Optimize probability function handling in LifeTable calculations

- Age_shifted revision

- Update TODO list with completed tasks and optimizations

- Update import paths to use 'src.lactuca' for consistency across test files

- Clarify comment on kwargs in ProbabilityCalculator

- Remove redundant validation check in _validate_float_sequence

- Replace functools.partial with direct function calls for improved performance

- Update TODO list with completed tasks and optimizations

- Simplify return statements by removing unnecessary float conversions in AnnuityCore and InsuranceCore classes

- Optimize numeric input validation by using copy=False in astype and replacing asarray with _as_f64_safe

- Rename parameter 't' to 'ts' in shifted method for clarity and consistency

- Optimize performance by replacing float conversions with item() and caching upper age limit in DecrementTable

- Update import paths to use 'src.lactuca' for consistency across test files

- Update TODO list with new tasks and optimizations for clarity and organization

- Update Chequeo_flujos.xlsx for improved data accuracy and organization

- Update Chequeo_flujos.xlsx for improved data accuracy and organization

- Update TODO.md to include refactoring tasks for dx_Tx_values and interpolation error fix

- Enhance decrement table methods for improved integer detection and interpolation

- Implement machine-precision integer detection in _is_int_machine function

- Improve integer detection in interpolation and validate generational tables in TableBuilder

- Create _EPS_INT constant for epsilon integer value to be used for integer detection more efficiently

- Replace machine-based integer detection with epsilon-based approach for improved accuracy and  robustness (_EPS_INT)

- Update life table loading to exclude specific test tables and improve clarity

- Rename and enhance _lx_at method for improved clarity and performance

- Deprecate _is_int_machine and improve integer detection in _validate_numeric_input

- Rename deprecated methods and improve lx value calculation in DecrementTable. _prob_life_interval method improved with validations and rounding for public path. _prob_decrement_interval improved with rounding for public path.

- Update TODO.md to include tasks for refactoring dx_Tx_values and implementing validation in TableBuilder

- Remove redundant comments for static type assertions in Config class

- Simplify type casting and improve readability in TableBuilder

- Streamline enumeration validation and improve cache management in Config module

- Simplify conditionals and improve readability in TableBuilder

- Improve readability and efficiency in InterestRate module by simplifying conditionals and optimizing cache management

- Update TODO.md with enhancements for InterestRate module and improve type handling

- Mark interest_rate.py as completed in TODO.md

- Add execution plan for splitting tables into subpackage

- Remove execution plan for splitting tables into subpackage

- Organize imports and improve readability in table_builder.py

- Move TableSource import to its own line for clarity

- Clean up imports and enhance readability in tables.py

- Update import path for read_table in test_16_TableSource.py

- Enhance TableSource class and add comprehensive tests for edge cases

- Improve documentation and formatting in config.py for clarity and consistency

- Improve readability of metadata validation in TableSource class

- Add debug scripts for LifeTable cache invalidation and validation

- Enhance DecrementTable for multi-instance dispatch and improve type safety

- Update import statements in config.py for improved type safety

- Optimize array operations in DecrementTable and LifeTable for improved performance

- Optimize suffix sum computation in DecrementTable and streamline w array extraction in LifeTable

- Remove deprecated probability validation functions and update tests for _assert_probs_in_unit_interval

- Consolidate numeric input validation and improve error handling in utils.py

- Enhance numeric input validation and streamline error handling in utils.py

- Simplify min/max checks and enhance numeric input validation in utils.py

- Introduce helper functions for array processing and streamline numeric input validation in utils.py

- Improve input normalization and validation in utils.py; streamline process handling in tests

- Enhance utility functions and improve validation in utils.py; update tests for consistency and edge cases

- Optimize numeric validation and error handling in utils.py; enhance performance with early exits and vectorized operations

- Optimize numeric validation and error handling in utils.py; streamline type checks and improve performance with cached type evaluations

- Improve performance in utils.py by optimizing min/max calculations and caching results; enhance error message handling with pre-computed sorted lists

- Simplify array creation by removing unnecessary dtype specifications in AnnuityCore and related classes

- Optimize TODO.md for clarity and consistency; streamline task descriptions for improved readability

- Clean up AnnuityCore calculations by removing redundant variables and improving clarity in present value computations

- Simplify growth calculations in AnnuityCore and ProbabilityCalculator by replacing power function with exponentiation operator

- Optimize calculations in AnnuityCore by improving ceiling operations and simplifying vectorized growth calculations

- Update TODO.md to clarify optimization tasks for core.py with a focus on mathematical precision and performance

- Replace power function with exponentiation operator in AnnuityCore and InsuranceCore for improved clarity

- Simplify term calculations in AnnuityCore by removing redundant multiplication

- Precompute error message prefix in _validate_numeric_input for efficiency

- Enhance optimization and clarity in TODO.md and utils.py by reducing redundancy and improving error handling

- Replace math.fabs with built-in abs for improved readability and performance

- Optimize boolean checks in _validate_numeric_input and _validate_cashflow_inputs for improved performance

- Move _make_prefix function for consistent error message prefix generation and optimize finite check in _validate_numeric_input

- Optimize finite checks in _assert_probs_in_unit_interval and streamline error message prefix generation in _validate_numeric_input

- Update copilot instructions with enhanced code optimization guidelines and improved file structure clarity

- Update optimization descriptions for clarity and precision in TODO.md

- Simplify numeric validation logic and enhance performance in validation helpers

- Improve error handling in _as_f64_safe and optimize boolean checks in _validate_numeric_input

- Enhance error reporting in _assert_probs_in_unit_interval for better clarity and performance

- Remove redundant assertions for type narrowing in DecrementTable and its subclasses

- Mark optimization completion for definitions.py and utils.py in TODO.md

- Update TODO.md to reflect optimization status for tables.py

- Add comments for clarity on validation and payment schedule logic in InterestRate class

- Improve code readability by formatting numpy array conversions in DecrementTable class

- Simplify DecrementTable class by removing unused TableClassLiteral and optimizing property access

- Update _FakeColumn class to use pandas-compatible interface for NumPy array handling

- Remove unused TableClassLiteral and VALID_TABLE_CLASSES from definitions.py

- Optimize DecrementTable class for performance and readability

- Update _get_probability_func to use Dispatcher enum for caller parameter

- Clean up DecrementTable class by removing unused imports and optimizing code structure

- Update TODO.md to reflect optimization status for tables.py

- Remove deprecated discount factor methods to streamline InterestRate class

- Enhance numeric input validation with overloads for better type handling

- Update TODO.md to mark interest_rate.py as optimized for performance and accuracy

- Optimize metadata handling and improve attribute assignment in TableSource class

- Update TODO.md to clarify optimization tasks and improve code prompts

- Simplify boolean coercion and enhance metadata handling in TableBuilder class

- Update file extension handling to enforce '.ltk' for TableBuilder files

- Remove obsolete actuarial table files from the repository notebooks\actuaruial_tables\

- Replace obsolete .tbl files with new .ltk files in actuarial_tables directory

- Update TableSource to load .ltk files and adjust file extension handling

- Update output path and filename extension handling for saved tables

- Enhance TableSource initialization and add file_path property

- Add tests for TableSource initialization with explicit paths and file path handling

- Update TODO.md with task completion indicators and path simplification notes

- Update TODO.md to clarify centralization plan and pending tasks

- Remove unused imports and dead code in core_helpers.py

- Import _MORTALITY_OFFSET and streamline mortality placement offset calculation

- Reorder imports in config.py for improved clarity

- Enhance LifeTable methods and improve mortality offset handling

- Update test cases to use consistent naming for interpolation map

- Mark task for updating _MORTALITY_OFFSET_MULT in InsuranceCore.discrete_precision as complete

- Remove unused _is_int_machine function from core_helpers.py

- Remove unused _is_int_machine function and update related tests for consistency

- Update TODO list to reflect completed tasks and clarify pending implementations

- Optimize division operation in LifeTable class for better performance and safety ex and Tx

- Remove deprecated Tx tests from DisabilityTable and ExitTable; consolidate Tx functionality tests in LifeTable

- Update TODO list with new tests for LifeTable and refactor tasks for DisabilityTable and ExitTable

- Optimize age and decrement column validations with single aggregations for improved performance

- Enhance Tx tests for consistency and edge cases; add new tests for monotonicity and numeric derivative

- Add generational validation test for user scenario; enhance cohort checks in existing tests

- Implement comprehensive tests for commutation symbols in LifeTable; cover various interest rates and edge cases

- Update TODO list with validation checks and testing priorities for LifeTable and related tables

- Refactor vx

- Optimize decimal handling and suffix sum calculation in LifeTable (commutation functions)

- Improve growth parameter validation in _validate_growth function

- Enhance survival calculations in ProbabilityCalculator for numerical stability

- Enhance reset functionality in Config class with detailed documentation

- Remove unused _J_RANGE_CACHE and _compute_fractionation_factor from LifeTable, enforce integer ages in commutation functions

- Remove unused discount cache and related methods from DecrementTable, optimize LifeTable cache initialization

- Remove unused _EPS_MACHINE import from core_helpers

- Remove deprecated generational formula functions and related constants from core definitions

- Update import paths for TableSource in test files to reflect new module structure

- Update import path for TableSource to reflect new module structure

- Rename decrement_scale to decrement_scale_factor for clarity

- Update actuarial tables to improve naming conventions and structure

- Rename lambda_scale_factor to mx_scale_factor in main function

- Streamline scaling factor retrieval and improve code readability

- Organize and add dummy life tables for testing purposes

- Remove legacy _generate_unisex_decrement method and enhance unisex rate generation logic

- Update dummy life table imports and filenames for clarity

- Update test cases to handle missing columns in unisex LifeTable and improve consistency checks

- Update TODO.md to clarify the blend generation in _generate_unisex_decrement and optimize function usage

- Implement new from_file method for loading TableBuilder instances from .ltk files

- Remove deprecated from_file_DEPRECATED method and introduce from_payload for improved TableBuilder instance creation

- Add to_payload method for serializing TableSource data and metadata

- Enhance logging and simplify table regeneration process in regenerate_ltk_files.py

- Standardize boolean values in test metadata across multiple test files

- Add analysis task for potential unification of data.py and regenerate_ltk_files.py

- Add table definitions with target filenames to data.py

- Simplify table imports by using TABLES from data.py

- Rename actuarial_age to act_age in benchmark_dates.py for consistency

- Enhance module docstring for clarity and update actuarial_age to act_age

- Rename actuarial_age to act_age for consistency in __init__.py

- Update import paths for core date functions

- Reorganize import statements for consistency and clarity

- Update import paths and remove deprecated test files for improved organization

- Reorganize import statements and improve formatting in life.py

- Update import statements in config.py and interest_rates.py for improved module organization

- Update import statements and class names for Insurance and Endowment engines

- Remove validators module and update documentation references for improved organization

- Update import statement for validators in life.py for improved organization

- Update import statement for validators in interest_rates.py for improved organization

- Update import statement for validators in test_04_config.py for improved organization

- Mark task as completed for reorganizing functions and definitions in core modules

- Add activate_venv.sh script and remove bash_script.sh for improved organization

- Update project documentation for improved clarity and organization

- Add from __future__ import annotations to multiple files for improved type hinting

- Update requirements.txt to specify package versions and remove comments

- Update dependencies in pyproject.toml to ensure compatibility and improve performance

- Update wheel build configuration and ensure unique output directories for compiled modules

- Replace index access with len() for improved readability in multiple files

- Allow CI to override CMake binary directory for improved build flexibility

- Enhance build directory management and diagnostics in CI workflows

- Streamline build tool installation for Windows and Linux/macOS in CI workflow

- Improve array slicing logic and enhance build directory management in CMake

- Enhance diagnostics and output directory management in CMake and update array slicing logic in tables

- Enhance CMake configuration with improved path normalization and install-time checks

- Enhance CMake configuration with improved path handling and install-time checks for built extensions

- Update ruff linting rules to include UP007 and expand per-file ignores

- Replace specific exception handling with general Exception in multiple modules

- Apply MSVC-specific compiler flags to suppress Cython-generated code warnings

- Add cleanup steps for build directories in CI workflow

- Enhance .pyi stub installation with existence check to prevent conflicts

- Streamline pre-build cleanup commands for Windows, Linux, and macOS

- Update Windows path separators to use forward slashes for consistency

- Fix malformed Windows paths in build and install prefixes for CMake compatibility

- Improve type annotation organization to avoid Cython warnings

- Relocate _build_metadata function for improved organization and clarity

- Reorganize _DecimalsConfig class for improved clarity and maintainability

- Remove parallel build configuration for cleaner CMake setup

- Enhance Cython source detection in wheel packaging workflow

- Add _DecimalsConfig class for improved decimal management in Config

- Enhance Cython source detection in artifact packaging workflow

- Reorganize _DecimalsConfig class for improved clarity and maintainability

- Remove parallel build configuration for cleaner CMake setup

- Fix formatting in public API documentation for consistency

- Update Python version in workflow to 3.14 for consistency

- Implement _DecimalsConfig class for improved decimal management in Config

- Update workflow to build wheels on macOS instead of Windows

- Update workflow to build wheels on Ubuntu instead of macOS

- Update workflow to include all Python versions from 3.11 to 3.14 for wheel builds

- Add names to artifact upload steps for clarity in workflow

- Simplify Python version comment in validate-wheels workflow

- Update tables path to use current working directory for user project root

- Update project notes to include additional resource link

- Enhance module docstrings for clarity and completeness across the Lactuca package

- Update .pyi stub generation to enhance security and proprietary code protection

- Enhance build optimizations and Cython security profiles for improved performance

- Remove redundant build type argument and debug symbol stripping for optimized binaries

- Limit build matrix to Windows and update Python version for wheel generation

- Enhance security compliance in stub generation and validation scripts

- Clean up comment formatting in validate-wheels.yml

- Improve docstring handling and inline comment detection in stub validation

- Disable packaging of Cython C++ sources in validate-wheels.yml due to absence of generated artifacts

- Optimize performance and readability in stub generation and validation scripts

- Add step to strip comments from __init__.py files for cleaner distribution

- Add per-file ignores for test files in linting configuration

- Optimize decimal precision settings for actuarial calculations

- Optimize list handling and performance in date calculations

- Optimize performance by reducing redundant calculations and leveraging Cython features

- Replace int type with np.intp for improved compatibility in decrement and life tables

- Enhance project notes with optimization tasks and context for static typing in Cython

- Reorganize imports and enhance code readability across multiple modules

- Replace len() with shape[0] for improved compatibility in InterestRate class

- Optimize payment time calculations with pre-calculated inverse for improved performance

- Update project notes with task completion and additional optimization details

- Enhance documentation with performance characteristics and Cython compilation details for date utilities

- Enhance module documentation with detailed descriptions of core components and usage examples

- Update project notes to clarify documentation tasks and next steps

- Update project notes to specify docstring continuation for __init__ in lactuca

- Enhance documentation for age calculation and time unit literals in base.py

- Enhance documentation for literal types and examples in base, helpers, and insurance modules

- Enhance DecrementTable for vectorial creation with detailed cohort handling and limits

- Clean up whitespace in generate_payment_times documentation

- Remove deprecated tests and enhance vectorial cohort functionality in LifeTable

- Enhance project notes with static typing and vectorization strategies for performance optimization

- Update API documentation and improve changelog structure

- Enhance project notes with automated documentation and changelog review workflow

- Update variable names in LifeTable and related documentation for consistency

- Streamline mortality data arrays and update start_age in GAM94_AA

- Streamline validation logic in _gen_discrete_improvement function

- Remove outdated architecture audit plan document

- Replace _gen_projected_improvement_cohort with _gen_projected_improvement in tests for consistency

- Enhance format_date documentation for clarity and detail

- Clean up __all__ exports in dates module for improved clarity

- Streamline imports and __all__ exports in engine module for clarity

- Remove commented lines for cleaner __all__ exports in tables module

- Simplify caching logic in TableBuilder for improved performance and clarity

- Clean up import statements and remove type ignore comments for improved clarity

- Update implementation status and checklist for CI/CD phases

- Remove unused import statements for improved clarity

- Remove unnecessary blank lines for improved code clarity

- Remove outdated comments and improve documentation clarity in base.py


### ✅ Testing

- Add comprehensive tests for _resolve_interest_rate function

- Add tests for lx interpolation with fractional ages near integers

- Add comprehensive tests for optimized _Tx_values functionality and performance

- Add unit tests for _lx_at and _lx_at_DEPRECATED to verify equivalence and performance

- Add unit tests for _check_probs function

- Add precision tests for commutation functions in LifeTable

- Add tests for omega boundary handling in qx calculations

- Enhance format_date tests with cross-format input coverage and edge cases

- Update save function to utilize fingerprint cache for payload hash


### ✨ Features

- Add centralized validation and definitions for actuarial calculations

- Implement _resolve_interest_rate function for validating and resolving interest rates

- Add centralized life inputs validation method (_validate_life_inputs) to LifeTable class and delete from utils.py

- Enhance InterestRate and LifeTable classes with improved validation and error handling

- Add comprehensive tests for InterestRate and LifeTable classes, including edge cases and boolean handling

- Improve numerical precision handling for scalar inputs in copilot instructions

- Update TODO.md with new tasks for unifying dispatch functions and improving type annotations

- Add bash script to activate virtual environment and run LifeTable tests

- Add performance benchmarking and tests for LifeTable methods, excluding specific table files

- Implement safe removal of np.clip in probability calculations with validation checks

- Implement centralized probability validation function to replace np.clip logic

- Replace np.clip with a helper in core.utils for stricter probability validation

- Add documentation for safe removal of np.clip in probability calculations

- Add task to replace np.clip with _check_probs in probability calculations

- Add probability validation method _assert_probs_in_unit_interval and refactor _prob_decrement_interval and _prob_life_interval

- Update TODO.md to include tasks for safe replacement of np.clip and validation in TableBuilder

- Add validation check for qx values in TableBuilder and update TODO.md

- Implement generational table formulas and update dispatcher in core.py

- Add centralized probability validation function to assert values in unit interval _assert_probs_in_unit_interval_assert_probs_in_unit_interval

- Add generational-specific validation checks in TableBuilder for decrement probabilities

- Refactor generational formula handling and probability validation in DecrementTable

- Enhance decrement validation by adding fail-fast checks for non-finite values in DecrementTable._modify_decrement

- Enhance TableBuilder validation for input data and generational probabilities

- Add generational validation tests for non-finite and out-of-range decrements

- Add tests for fractional qx calculations and ensure monotonicity in interpolation

- Add tests for extreme multiplier overflow and non-finite values in modify_qx

- Update documentation on safe removal of np.clip in probability calculations

- Implement validation for generational decrements in TableBuilder

- Remove np.clip for tpx and px_matrix in MuApproximator and ProbabilityCalculator for improved numerical stability

- Enhance _modify_decrement method with detailed docstring and improved validation for modifications

- Refactor LifeTable tests to use _make_life helper for improved clarity and consistency

- AnnuityCore.discrete_simplified refactor payment_size and payment_frac. Optimization of interpolation coefficients calculation. _joint_survival_udd_yeawise_func added qx_flat = np.asarray(qx_flat, dtype=np.float64) due to error when DecrementTable_prob_decrement_interval returns scalar and not array.

- Add high-precision tests for annuities, endowments, and insurances with detailed configurations

- Update tests to use deprecated _check_probs function for compatibility

- Update Chequeo_flujos.xlsx with new data

- Add Pydantic-based configuration management with TOML support

- Enhance Config module with performance improvements, new methods, and comprehensive tests

- Add allowed mortality placements and module-level constant for offset lookup

- Add tests for deferred annuity and insurance equivalences with commutation functions

- Implement commutation functions for improved actuarial calculations

- Add documentation notes for third-party license management services and project distribution

- Add centralized validation for integer ages in commutation functions and implement continuous versions of Tx, Lx, and ex

- Add pytest fixture to automatically reset Config singleton state after each test

- Enhance TODO documentation with additional implementation details for commutation functions and maintainability improvements

- Enhance LifeTable methods for continuous age calculations and validation, ensuring proper handling of fractional ages. Functions Lx, Tx, ex and their _continuous counterparts now support non-integer ages with improved accuracy and validation checks.

- Update TODO documentation to include numerical integration for continuous age calculations

- Add integer detection and tolerance guidelines for commutation functions, including centralized helpers and performance considerations

- Add integer and fractional detection helpers with optimized performance and detailed documentation

- Optimize integer and fractional age detection in commutation functions for improved performance

- Add comprehensive unit tests for integer detection helpers to ensure correctness and consistency

- Update TODO documentation to reflect review of pyliferisk functions and validation of commutation symbols

- Enhance input normalization and precision handling guidelines in copilot instructions

- Enhance integer detection tolerance documentation and adjust epsilon scaling for improved precision

- Add sub module tables splitting tables.py into multiple files and adjust imports accordingly. Additionally include table_builder.py and table_source.py into tables submodule.

- Add read-only table property to access TableSource metadata

- Enhance summary method to include table metadata and examples for DisabilityTable, ExitTable, and LifeTable

- Add tests for table property access in DisabilityTable, ExitTable, and LifeTable

- Add property to DecrementTable for accessing associated TableSource metadata

- Refactor decrement combination methods and update allowed sexes

- Enhance valid sexes extraction and validation for lambda columns

- Refactor DecrementTable to support unisex blending and improve table combination logic

- Refactor DisabilityTable to enhance modify_ix method and support unisex blending

- Refactor ExitTable to enhance modify_ox method and support unisex blending

- Refactor LifeTable constructor to support unisex blending

- Update binary actuarial tables for improved compatibility

- Add script creation for tables and refine table combination logic

- Remove license-expression from pyproject.toml

- Update actuarial tables with new binary data files

- Add script to regenerate .ltk files with updated schema (omega, sex_independent)

- Remove 'age' column and add 'omega' and 'sex_independent' metadata fields

- Add omega and sex_independent fields, remove age column, and enhance validation

- Update tests to remove age column, add omega and sex_independent fields, and validate omega metadata

- Update TODO.md to include checks for generational table blends and sex_independent table creation

- Add DummyLIFE_1S.ltk actuarial table

- Add DummyLIFE_1S life table with static probabilities and omega field

- Update allowed sexes to include 'u' for unspecified gender

- Update metadata structure in TableSource to include parsed values for valid_sexes, sex_independent, and generational

- Enhance DecrementTable to support unisex blending and case-insensitive sex input

- Update summary method in DisabilityTable to include comprehensive table metadata and sample ix values

- Update summary method in ExitTable to include comprehensive table metadata and sample ox values

- Update summary method in LifeTable to include comprehensive table metadata and sample qx values

- Update static table retrieval to exclude single-sex validation table for continuous interpolation tests

- Add dummy_life_1S_data to regenerate_ltk_files script for new schema

- Update TODO.md to reflect completion of generational blend and sex-independent table checks

- Update actuarial tables to reflect new schema changes

- Enhance generational formula descriptions for clarity and extensibility

- Correct naming of generational formula types in exports

- Remove generational formula references and update related logic for sex-independent calculations

- Remove age-dependent generational formula from actuarial tables for sex-independent calculations

- Remove generational formula attribute and related property for sex-independent calculations

- Remove generational_formula assertions and update related tests for non-generational tables

- Update TODO.md with additional tasks for generational formula types and international tables

- Update TODO.md with new tasks for interpolation efficiency and independent table functions

- Add script to regenerate .ltk files with new schema

- Add new binary actuarial tables DummyLIFE_1Sf.ltk and DummyLIFE_1Sm.ltk

- Add script to regenerate .ltk files with updated schema

- Add comprehensive tests for Tables public API and multi-install helpers

- Add case-insensitive filename conflict resolution in TableBuilder

- Refactor table installation logic and improve payload validation

- Update binary file for DAV2004R table data

- Remove obsolete binary files from petecandel notebook directory

- Update binary file for DAV2004R table data

- Enhance DataFrame creation with error handling for invalid data shapes

- Refactor table entry handling and improve error messaging in data.py

- Enhance table installation tests with strict mode error handling and shape validation

- Improve blending logic in DecrementTable for consistent rounding and precision

- Update unisex blend tests to use rounded cohort-adjusted rates for accuracy

- Update TODO.md to reflect completed tasks and clarify table integration

- Update TODO.md to reflect completed tasks and clarify table functions

- Add pandas as a dependency in pyproject.toml and requirements.txt

- Enhance date normalization and calculation functions; rename actuarial_age to act_age for consistency

- Add date_format configuration with validation and default value

- Add tests for date_format configuration and act_age functionality

- Add documentation for uploading packages to PyPI using Cython and GitHub Actions

- Update module docstring to enhance clarity on core functionalities and public submodules

- Enhance module docstring to provide detailed overview and usage examples for actuarial table utilities

- Expand module docstring to include detailed descriptions of functionalities and design goals

- Add context manager for date format configuration in tests and enhance date normalization tests

- Update documentation for PyPI publishing process and security considerations

- Update TODO.md with new tasks for communication and package distribution

- Add valid date formats and corresponding allowed formats

- Add format_date and FormatDates to public API for enhanced date formatting

- Add format_date and FormatDates to enhance date formatting capabilities

- Enhance format_date functionality with additional tests and mixed type support

- Reorganize function imports and definitions for better module structure

- Add time_diff function to public API for enhanced duration calculations

- Add time_diff to __all__ for public API exposure

- Update documentation guidelines for English docstrings and NumPy format compliance

- Add comprehensive documentation guide for automated API documentation generation using MkDocs and NumPy style

- Add valid time units for time_diff function to enhance duration calculations, enhance date quarter formatting with prefix and suffix options, and improve leap year detection for multiple years.

- Add comprehensive tests for time_diff function covering various units and edge cases

- Add InsuranceEngine for life insurance calculations

- Implement FASE 0.0 for production code preparation

- Add comprehensive API documentation and user guides. FASE 1

- Add scripts for generating and validating .pyi stub files for proprietary code protection

- Add security validation for .pyi stub files to protect proprietary code

- Add script to strip comments from __init__.py files for cleaner distribution

- Add CI/CD workflows for documentation and release processes

- Add new actuarial tables for PASEM and PER 2020

- Add new GAM data files for various demographics

- Add new PEAI2007 IAP actuarial tables for individual and collective data

- Add PEAI2007 IAP actuarial tables for individual and collective disability data

- Update TablasMej2.xlsx with new data

- Add new GAM actuarial tables (GAM71, GAM83, GAM94_AA)

- Add GAM actuarial tables (GAM71, GAM83, GAM94_AA) and implement discrete improvement formula

- Add tests for decrement_multiplier functionality and GAM 94 discrete improvement formula

- Update GAM actuarial tables for 1971 and 1994 with new data files

- Include start_age in metadata and validation for actuarial tables

- Update GAM actuarial tables with new data files for 1971, 1983, and 1994

- Add new actuarial tables for GAM, PER, and other models

- Add original_start_age to metadata and validation in actuarial tables

- Include start_age in DecrementTable methods and update age range representation

- Adjust age range in LifeTable to start from 0

- Add original_start_age property and update data viewing methods in TableSource

- Update tests for GAM94_AA to validate age 0 padding and age 1 data

- Update actuarial tables with new binary data files

- Update generational formula validation to allow non-negative improvement factors

- Update decrement table to use start_age instead of original_start_age in documentation

- Add start_age property to TableSource for original table definition

- Replace original_start_age with start_age in LifeTable and related tests

- Update actuarial tables with new binary file versions

- Remove original_start_age from metadata model and update start_age description

- Sanitize mx values in DecrementTable to ensure strictly positive inputs

- Replace original_start_age with start_age in TableSource for improved clarity

- Update tests to reflect removal of original_start_age and ensure start_age is correctly handled

- Update sync-public workflow to remove push trigger and retain manual dispatch only

- Add select and select_period fields to metadata model and table builder

- Add duration support for select-ultimate tables in DecrementTable

- Add duration parameter to DisabilityTable initializer

- Add duration parameter to ExitTable initializer

- Add duration parameter to LifeTable initializer

- Add select and select_period support to TableSource

- Enhance mx column validation and add checks for discrete improvement

- Implement per-duration mx column validation and enhance mx column retrieval logic

- Update GAM Excel files for 1994 Scale AA - Female and Male

- Add GAM94_AA mortality table with discrete improvement factors

- Add new actuarial tables for 2020 (B_M, CB_H, MI_H, MI_M, RV_M) and update Dummy_qx0

- Implement Chilean actuarial tables and projected improvement formula

- Add projected improvement formula and update generational formula types

- Add grid_years support and expand year-keyed columns for projected improvement

- Enhance DecrementTable to support projected improvement with calc_year validation

- Add calc_year parameter to DisabilityTable initialization

- Add calc_year parameter to ExitTable initialization

- Add calc_year parameter to LifeTable initialization and update Tx calculations

- Add grid_years support for projected improvement tables and enhance life table validation

- Update project notes with new to-do items for Chilean tables and code optimization

- Update actuarial tables with new binary data files

- Update improvement rate terminology and enhance documentation for clarity

- Add new selected ultimate actuarial tables and remove deprecated ones

- Update decrement table to use 'mi' terminology and improve validation logic

- Update tables path to point to src directory and improve file reading logic

- Update type hints and improve rate calculations in InterestRate class

- Add taxonomy and classification for actuarial tables in project notes

- Update input format from `mx_*` to `rf_*` and enhance projected improvement calculations

- Update actuarial tables with new binary data files

- Add regex validations for MI column names and enhance error handling in TableBuilder

- Add calc_year property with validation and warnings in DecrementTable

- Optimize LifeTable initialization and cache management

- Enhance select_rf detection with regex and improve error handling for select_period

- Add tests for calc_year and unisex_blend setters in LifeTable; enhance omega validation in TableBuilder

- Update multiple actuarial tables with new binary data

- Update project notes for Chilean tables and cohort projections

- Rename and update projected improvement function for cohort-diagonal projection

- Add mi_structure inference and update TableBuilder metadata

- Rename and update projected improvement handling for cohort-based projections

- Remove unused calc_year parameter from DisabilityTable initialization

- Remove calc_year parameter from ExitTable initialization

- Remove calc_year parameter from LifeTable initialization

- Add mi_structure metadata handling for generational tables

- Update tests to use mi_by_duration for per-duration improvement routing

- Update comments and variable names to reflect mi_by_duration usage in DecrementTable

- Rename _select_rf to _mi_by_duration for clarity and update related documentation

- Enhance DecrementTable with pass-through properties for direct metadata access

- Enhance summary and taxonomy lines for improved metadata representation

- Improve test assertions for sex-specific columns and enhance summary output in unisex blend tests

- Add comprehensive architecture audit plan with identified issues and implementation steps

- Add layer mapping for Lactuca architecture in project notes

- Add comprehensive refactor plan for Lactuca architecture audit

- Enhance Cython guidelines for class definitions, attribute initialization, and caching practices

- Add sequence normalization and date formatting helpers in core.py

- Create independent copy of pv_vec_raw to prevent aliased keys in AnnuityEngine

- Simplify return type descriptions in EndowmentEngine documentation

- Defer imports of CubicSpline and gaussian_filter1d to reduce cold start time

- Enhance configuration management and improve validation logic across modules

- Update configuration access in tests to use Config singleton directly

- Add prompt for deep architecture audit and future implementation guidelines

- Enhance performance guidelines with vectorized validation and thread-safe singleton pattern

- Update refactor plan with completed tasks and improved threading safety in Config

- Improve thread safety in Config singleton with double-checked locking

- Enhance interest rate validation to ensure finiteness and improve performance

- Update binary actuarial table DAV2004R_SelUlt_1o.ltk

- Enhance duration handling in DecrementTable for select tables

- Update project notes with testing examples and new section on projected improvement factors

- Update TODO.md with completed tasks and add caching for probability functions

- Enhance generational_formula_type handling and add projected_improvement logic

- Add tests for Select-Ultimate with projected_improvement in Gap 9

- Implement projected_improvement logic in _compute_select_decrement

- Update TODO.md with completed tasks and remove discarded items

- Rename tbls_others to tables_others for consistency in LifeTable methods

- Add standalone functions to the functional API in __init__.py

- Update test files to use consistent parameter naming and add functional API tests

- Update project notes to reflect changes in table naming conventions

- Add per-file ignores for functional.py in linting configuration

- Add new documents for CNSF 2000-G, CNSF 2000-I, and archive zip file

- Add implementation plan for GrowthRate class in PLAN_GrowthRate.md

- Add new Excel documents t15006.xlsx and t15007.xlsx to Mexicanas directory

- Add link to Mexican tables in TODO.md

- Add growth rate parameter to AnnuityEngine and InsuranceEngine for enhanced growth calculations

- Integrate GrowthRate parameter into LifeTable for enhanced growth calculations

- Add GrowthRate class for modeling growth rate scenarios in life annuities and insurance payments

- Integrate GrowthRate parameter into InterestRate class for improved growth calculations

- Import GrowthRate class into __init__.py for public access

- Add _resolve_gr function for validating and resolving GrowthRate instances

- Add comprehensive tests for GrowthRate class and _resolve_gr helper

- Update implementation plan for GrowthRate class, marking completion and refining test coverage

- Mark implementation of GrowthRate class in TODO.md as complete

- Refactor AnnuityEngine to use GrowthRate class for growth parameters

- Remove deprecated VALID_GROWTH_TYPES and associated comments from base.py

- Refactor InsuranceEngine to use GrowthRate object for growth parameters

- Remove growth rate and type parameters from LifeTable class, updating to use GrowthRate model

- Add GrowthTypeLiteral and VALID_GROWTH_TYPES for actuarial growth options

- Update ax, äx, axy, äxy, axyz, ajoint, and Afirst functions to use GrowthRate for growth parameters

- Refactor GrowthRate class to improve validation and update growth type handling

- Refactor InterestRate class to remove growth rate and type parameters, integrating GrowthRate model

- Remove growth validation functions and parameters from cashflow inputs

- Add validation to ensure 'gr' and 'cashflow_amounts' are mutually exclusive in LifeTable

- Add validation to ensure 'gr' and 'cashflow_amounts' are mutually exclusive in InterestRate class

- Enhance GrowthRate documentation with detailed examples for piecewise growth scenarios

- Enhance GrowthRate tests with additional scenarios and validations

- Update TODO.md to include implementation details for fractional_ts_interpolation helper

- Add growth rate conventions documentation and update index for fractional time shifts

- Create comprehensive plan document for fractional_ts and update related validation messages

- Update validation messages for fractional shifts to clarify actuarial conventions

- Enhance documentation for growth rate conventions and fractional time shifts in LifeTable

- Add documentation reference for anniversary convention and fractional-ts policy in GrowthRate class

- Add prompt for deep architecture audit and Cython usage recommendations in project notes

- Update TODO with implementation details for fractional_ts_interpolation helper and discard related plan reference

- Update section titles to uppercase for consistency in project notes

- Update TODO list with completed tasks and improve function typing using mypy

- Optimize leap year calculation and improve exception handling in date functions

- Refactor error handling in TableBuilder using a dedicated validation helper

- Improve type handling and performance in DecrementTable methods

- Update DisabilityTable methods to enforce blocking with explicit parameters

- Update ExitTable methods to enforce parameterized blocking for qx, ix, ex, and modify_qx

- Add life survival and probability functions with parameterized blocking

- Optimize enumeration validation in _ConfigModel using module-level constant

- Enhance NumPy compatibility in _as_f64_safe by using dynamic copy behavior

- Streamline InterestRate class by simplifying attribute access and initialization

- Refine optimization notes for maximum execution speed and precision in project documentation

- Update TODO.md with completed tasks and new action items for project organization

- Rename and update _gen_projected_improvement_cohort function for clarity and consistency

- Rename _gen_projected_improvement_cohort function to _gen_projected_improvement for clarity

- Rename _gen_projected_improvement_cohort function to _gen_projected_improvement for clarity

- Add sphinxcontrib-mermaid dependency for enhanced documentation support

- Add comprehensive Sphinx documentation plan for project structure and phases

- Update documentation audit status and complete API page splitting

- **prompts**: Add new prompts for Conventional Commits, new table addition, and release checklist

- Add compact actuarial age aliases for improved usability

- Implement tiered cashflow amounts and GrowthRate utility functions

- Implement webhook event handlers and Keygen license management (B.3/B.7)

- Implement hardware fingerprint control for trial abuse prevention

- Implement trial license creation and validation in webhook

- Implement trial license activation and webhook handling

- Integrate Resend for trial license email delivery and update endpoint configurations

- **webhook**: Enhance trial email functionality with language detection and dynamic content

- **licensing**: Implement license verification and activation system

- **activation**: Implement HMAC-based activation token verification and enhance license validation flow

- **activation**: Add PEP 562 module-level __setattr__ guard (C.9c)


### 🐛 Bug Fixes

- Correct import path for EndowmentCore in test file

- Update tpx function calls to use 't' parameter for consistency in annuity tests

- Add explicit cashflow times adjustment flag in AnnuityCore

- Correct parameter in ax_aex_vitalicia_consistency test for whole life annuities

- Adjust payment size calculation for whole-life insurance to include final death payment

- Update TODO for commutation symbols tests and configuration validation

- Update tests to use _EPS_INT for consistency in tolerance checks

- Adjust boundary checks in DecrementTable to improve robustness and precision. In _prob_decrement_interval _EPS_MACHINES is eliminated.

- Improve handling of ages at or beyond omega in DecrementTable calculations (_prob_decrement_interval)

- Update age validation in DecrementTable to allow x > w according to actuarial conventions

- Update TODO for testing commutation symbols and improve clarity

- Improve error message formatting for invalid sex in DecrementTable

- Update Germany Annuities documentation and remove unnecessary whitespace

- Correct spelling in TODO.md and update task descriptions for clarity

- Correct LifeTable instantiation example in documentation

- Update LifeTable instantiation examples to use correct table name format

- Correct LifeTable instantiation examples to use proper table name format

- Correct LifeTable instantiation example in README.md

- Update Tables.install calls to use lists for source specifications

- Update TODO.md to include specific Latin American countries for international tables

- Correct spelling and accentuation in descriptions of actuarial tables

- Update mx validation to require strictly positive values

- Correct rounding logic and update terminology in disability, exit, life, and source tables

- Correct spelling in project notes and enhance clarity on implementation requirements for projected improvement

- Update growth_conventions references in documentation for consistency

- Update documentation link for growth conventions to point to user guide

- Update doctests in _validate_x_and_x0 for clarity and consistency

- Update documentation to reflect changes in projected improvement function

- Correct formatting in generational tables documentation for clarity

- Improve formatting in LifeTable documentation for consistency

- Improve documentation clarity in Config class for validation and error handling

- Enhance documentation examples and formatting in InterestRate class for clarity

- Standardize section headings in project notes for consistency

- Enhance multi-version structure creation in documentation build script for cross-platform reliability

- Export config=Config() singleton alias from lactuca.__init__

- Correct tables path in configure_tables_path fixture and remove unnecessary src prefix

- Update path for actuarial tables in configuration

- Update documentation for pure endowments and force_integer_ts parameter

- Normalize base_path upfront in process_init_files and optimize print statement

- Update encoding to utf-8-sig for BOM handling in generate_stub_file function

- Remove invisible characters from docstring in core.py

- Correct formatting comment in data_tables.py

- Escape embedded triple-quotes in docstrings to prevent premature closing

- Simplify vercel.json to use automatic API routing

- Restore builds in vercel.json without explicit routes

- List api handlers individually in vercel.json builds

- Add webhook/pyproject.toml to isolate from root uv.lock

- Use absolute dest paths in vercel.json routes

- Switch to Flask app.py entrypoint for Vercel Python detection

- Add flask to webhook/pyproject.toml dependencies

- Add root health route and both /webhook and /api/webhook paths

- Update webhook architecture and implement Flask entrypoint in app.py

- Update verification status for Phase B tests in licensing documentation

- Update subscription event handlers to return 200 status for missing licenses

- Configure Vercel to serve Flask app from app.py

- **webhook**: Use GET /users/{email} to retrieve existing Keygen user

- **webhook**: Move resend import inside try/except to prevent startup crash

- **webhook**: Use metadata[user_email] filter for trial duplicate check

- Add resend to webhook pyproject.toml, remove redundant requirements.txt

- Update support email address in documentation and code

- **config**: Add __deepcopy__ to Config singleton to support LifeTable.copy()


### 📚 Documentation

- Update TODO.md to include tasks for validating generational decrements in TableBuilder

- Update documentation on safe removal of np.clip in probability calculations

- Update TODO.md with additional test for decimal configuration

- Update TODO.md with additional testing notes for commutation symbols

- Enhance documentation for Config.reset() method to clarify singleton behavior and usage examples

- Enhance documentation for continuous annuity calculations, clarifying payment timing and integration methods

- Mark task to divide tables.py for improved maintainability

- Add project_notes.md for updating project requirements

- Update project notes with new TODO item for GitHub workflow error explanation

- Update copilot instructions to specify type hinting requirements and rationale

- Update copilot instructions with Python 3 exception syntax and modern type hints guidelines

- Add module-level docstring for TableSource class

- Add comprehensive documentation instructions for the Lactuca project

- Enhance documentation clarity and consistency across multiple guides

- Clean previous build before Sphinx documentation generation

- Improve formatting and clarity in config and growth_rates documentation

- Add 92series.xlsx to UK documentation

- Add whitespace for improved readability in helpers.py

- Clarify documentation for force_integer_ts and insurance methods in life.py

- Enhance documentation for force_integer_ts behavior in config.py

- Add has_growth_rate parameter to InterestRate initialization for clarity

- Add has_growth_rate parameter to _validate_shift_deferment_and_duration for improved warning control

- Update projections documentation for clarity and support of multiple formulas

- Update UK 92 series Excel file with latest data

- Add AM92/AF92 mortality table with detailed description and data

- Enhance documentation for AM92/AF92 tables and improve clarity across various sections

- Add AM92_AF92 actuarial table as a new binary file

- Update decrement column documentation and validation for select durations

- Rename AM92_AF92_SelUt to AM92_AF92 and streamline description in data_tables.py

- Update duration validation and documentation in DecrementTable class

- Add start_duration property and update related documentation in TableSource class

- Enhance error message for duration validation in DecrementTable class

- Enhance documentation for TableSource attributes and metadata structure

- Add __setattr__ method to _DecimalsConfig class to enforce read-only attribute assignment

- Enhance descriptions in mortality tables to include DGSFP regulatory framework

- Update LifeTable documentation for global decimal precision configuration

- Add tests for _DecimalsConfig proxy to enforce read-only behavior

- Correct encoding issues in Chilean Generational Life Tables comments

- Enhance DecrementTable property setters to clarify cache invalidation and no-op behavior

- Clarify caching behavior for InterestRate in LifeTable documentation

- Update Config documentation to clarify TOML section handling and remove unused sidecar lock implementation

- Add tests for dirty-checks and bug fixes in DecrementTable setters

- Add implementation details for dirty-checks and bug fixes in DecrementTable setters

- Update Mexicanas and UK documentation with new and modified Excel files

- Update PEAI2007_IAP_Ind.ltk binary file

- Remove is_generational_life method and update public scale factor handling in TableBuilder

- Remove is_generational_life tests for cleaner codebase

- Clarify life expectancy methods in LifeTable documentation

- Enhance documentation for exponential and linear improvement methods

- Update documentation for select-duration columns and expand functionality

- Enhance tests for generational formulas and TableBuilder functionality

- Add task to review and propose improvements for markdown documentation

- Add bugfix tracking for DecrementTable._modify_decrement with detailed summary and phase plan

- Add detailed plan for redesigning `table_combination` API in DecrementTable

- Update documentation to replace 'DecrementCombination' with 'Decrement types' in type literals

- Update documentation to reflect changes in decrement combination methods and types

- Enhance DecrementTable documentation and improve cache handling

- Update DisabilityTable documentation for table_combination and modification behavior

- Update ExitTable documentation for table_combination and modification behavior

- Enhance Modifying Decrements documentation for clarity and completeness

- Enhance DecrementTable and related documentation for clarity and detail

- Enhance DecrementTable documentation for internal state and modification tracking

- Update summary method documentation in DisabilityTable for clarity and detail

- Update summary method documentation in ExitTable for clarity and detail

- Enhance summary method documentation in LifeTable for clarity and detail

- Add new actuarial tables for DummyEXIT and DummyLIFE categories

- Enhance documentation for DecrementTable, ExitTable, and DisabilityTable for clarity and completeness

- Add extended dummy tables catalog with comprehensive coverage and testing

- Add pricing structure and validation strategy for subscription model

- Update import statements to use importlib for accessing lactuca.config module

- Enhance documentation for new actuarial tables and clarify usage in user guide

- Update InterestRate class to use Scalar and ScalarSequence for type hints

- Add tests for sn and i_m functions in InterestRate class

- Enhance user guide for commutation functions with detailed explanations and examples

- Enhance lx_interpolation documentation with functional-style examples and clarify usage

- Enhance interest rates guide with additional examples and clarify usage of methods

- Enhance documentation with new pricing structure, subscription model validation, and improved multi-version support

- Update GrowthRate documentation with new methods, enhanced validation, and improved string representation

- Add tracking document for `__str__` and `summary()` implementation in `GrowthRate`

- Enhance user guides for InterestRate and GrowthRate classes with detailed examples and clarifications

- Add tests for GrowthRate.__str__ and summary() methods, including edge cases and annotations

- Update growth rates

- Add plan for vectorial interest rate support in LifeTable

- Clarify handling of legacy kwargs in DecrementTable and update LifeTable usage example

- Add __new__ method to LifeTable for flexible interest rate handling

- Clean up imports and format hash calculation in Config class

- Add vectorial interest rate tests for LifeTable functionality

- Update DisabilityTable to clarify blocked decimal properties and improve formatting

- Enhance ExitTable decimal configuration to clarify blocked properties and improve lookup efficiency

- Enhance validation messages in LifeTable for integer age checks and suggest continuous methods

- Add reference to decimals rounding in configuration guide

- Clarify usage of Config singleton and update examples for decimal settings

- Enhance numerical precision documentation with detailed strategy overview and clarifications on integer detection

- Add vectorial interest rate support in LifeTable and clarify legacy kwargs handling in DecrementTable

- Enhance tests for decimals handling in LifeTable, DisabilityTable, and ExitTable, ensuring proper error messages and access restrictions

- Update functional API documentation and improve cashflow parameters

- Add tracking documents for improvements in generate_payment_times, including bug fixes, optimizations, and expanded documentation

- Clarify documentation for cashflow dictionary structure and update present value calculation in InsuranceEngine

- Standardize parameter names for cashflow in LifeTable class

- Update example for generate_payment_times to include selected_periods parameter

- Standardize parameter names for cashflow in ax, axy, axyz, and ajoint functions

- Standardize parameter names for cashflow in InterestRate class

- Enhance generate_payment_times documentation and improve parameter handling

- Add regression test for discrete_simplified return flows with m > 1

- Add comprehensive tests for generate_payment_times covering various schedules and edge cases

- Standardize parameter names for cashflow in ax and a functions

- Enhance documentation for return_flows in AnnuityEngine

- Update integration points for insurances and pure endowments in calculation modes

- Clarify returned dict structure for discrete_precision with m=1 in inspecting_cashflows

- Update functional API documentation and standardize cashflow parameters across multiple classes and functions

- Add comprehensive documentation agent for writing, editing, and reviewing Lactuca documentation

- Enhance documentation assistant roles and responsibilities for Lactuca project

- Enhance documentation for AnnuityEngine and clarify deferred insurance formulas

- Clarify calculation notes in LifeTable regarding deferment and duration

- Improve documentation for payments_size and payments_frac parameters in AnnuityEngine

- Update DecrementTable documentation to reflect new _LX_RADIX value

- Update _flatten_loaded_config to clarify key mappings for calculation, calendar, and mortality sections

- Update test configurations to include calendar section for validation and loading

- Add implementation details for non-standard payment frequencies (m=14, 24, 26)

- Add comprehensive user guide improvement plan with detailed phases and actions

- Enhance documentation for DecrementTable and configuration mappings

- Update payment frequency options in anniversary_dates and act_age functions

- Expand payment frequency options and descriptions in base.py

- Update payment frequency options in generate_payment_times function

- Enhance payment frequency validation documentation for clarity

- Update payment frequency validation and acceptance in tests

- Update performance optimization notes and add prompt for user guide improvement

- Add examples to date-related functions for improved clarity

- Update examples in tables to reflect new LifeTable structure and improve clarity

- Update user guide to include detailed explanations for prospective reserves and payment frequency

- Enhance documentation with detailed examples for annuity and insurance calculations

- Enhance documentation with detailed examples for GrowthRate and InterestRate classes, and update usage in functional examples

- Enhance documentation and user guide for payment frequency options and validation

- Update actuarial docstring guidelines to clarify banned sections and auditing process

- Enhance API documentation with new sections for Config, Date Utilities, and various table classes; add examples and improve clarity throughout

- Update documentation to remove references to Spanish actuarial organizations and streamline language for international clarity

- Update documentation to remove references to Spanish actuarial organizations and ensure compliance with international standards

- Update documentation to remove references to Spanish actuarial practices and ensure alignment with international standards

- Add pytest configuration instructions to project notes

- Enhance 'See Also' guidelines for Unicode and method references in docstrings

- Enhance API documentation with new sections and examples; update actuarial docstring guidelines for clarity

- Update actuarial docstring guidelines for subclass API pages and inherited members

- Add comprehensive docstring auditor agent for src/lactuca/ Python files

- Add comprehensive docstring audit prompt for Python modules, including automated and manual workflows

- Add docstring auditor agent and audit-docstrings prompt for systematic validation

- Update customization tracker with new /audit-docstrings prompt for systematic review

- Enhance documentation for DisabilityTable, ExitTable, and LifeTable with inherited members and methods

- Enhance API documentation and improve Sphinx configuration for better clarity and usability

- Update actuarial docstring guidelines to enhance clarity and consistency in parameter references and examples

- Update actuarial docstring guidelines and enhance documentation for subclass API pages and inherited members

- Enhance InterestRate class docstrings for clarity and consistency

- Add text accuracy and redundancy audit step to docstring auditing process

- Update actuarial docstring guidelines for method and attribute references in narrative text

- Update actuarial docstring guidelines and enhance InterestRate class documentation for clarity and consistency

- Enhance docstring references for DecrementTable and its subclasses for clarity

- Add guidelines for using Pattern D for Unicode method names in docstrings

- Enhance docstring auditing process and update actuarial guidelines for clarity

- Refine docstrings for clarity and consistency across TableBuilder methods

- Update example in ExitTable docstring to include unisex_blend parameter

- Improve docstring formatting and consistency across TableSource class

- Update docstrings for GrowthRate class to enhance clarity and consistency

- Enhance docstring clarity and consistency across InterestRate class methods

- Add tests for shifted() numerical coherence with apply_from_first=True

- Update pytest configuration to enhance test options and add benchmark marker

- Remove obsolete pytest configuration file

- Refine documentation for TableBuilder class and related functions

- Update

- Update docstrings

- Refine documentation for generate_payment_times function

- Remove deprecated test for payload hash verification

- Update utility functions section to enhance clarity on payment grid helpers

- Update table type references to include class links for clarity

- Enhance docstrings for clarity and consistency across various classes and methods

- Update documentation for actuarial age aliases and remove deprecated aliases

- Update example output to reflect float return type for age_last_birthday function

- Update import statements and enhance docstrings for date utilities

- Add compact actuarial age aliases to top-level namespace

- Enhance documentation for Config class and improve clarity of method descriptions

- Enhance date utilities documentation with detailed capabilities and compact aliases

- Enhance configuration documentation by adding thread-safety note for set_many method

- Update short-form aliases in dates guide to avoid shadowing built-in names

- Refine documentation for TableBuilder class and enhance clarity of related functions

- Add implementation document for Config.set_many thread-safety fix

- Add test for thread safety in Config.set_many method

- Add tests for anniversary_dates function with various intervals and edge cases

- Clarify support for pandas and polars as required dependencies and add DateLike type alias

- Update documentation to clarify native integration with pandas and polars

- Refine documentation for pandas and polars integration, removing conditional notes

- Enhance API documentation with additional details on configuration, date types, and table functionalities

- Add tests for vn() function to validate rate conditions and error handling

- Enhance validation for interest rates in discount factor calculations

- Add documentation for proprietary actuarial API distribution and implementation guide

- Add note on ValueError for vn() function with invalid rates

- Update contact information for Alberto Aragoneses

- Add tests for generate_payment_times function covering various scenarios

- Add tests for GrowthRate.amounts and tiered_amounts functions

- Add documentation for tiered cashflow amounts function

- Add examples for generating cashflow amounts and tiered cashflows

- Enhance documentation and add tests for cashflow utilities

- Add amounts method for cashflow generation in GrowthRate class

- Update module documentation and add tiered_amounts function for cashflow assignment

- Update CI/CD implementation plan with pre-0 audit and stub publishing details

- Enhance CI/CD documentation with implementation references and updates

- Update payment frequency documentation and examples for clarity

- Update CI/CD implementation plan with validation results and corrections for wheels build

- **impl**: Update IMPL_distribution_licensing with C.9c and deepcopy details


### 🧹 Miscellaneous

- Update TablasMej2.xlsx with new data

- Remove Pydantic dependency from requirements.txt

- Update DummySD2015Gen.ltk binary file

- Add external folder to .gitignore for temporary files of test_pypi_private package

- Update TablasMej2.xlsx with new data

- Update project notes with manual documentation review tasks and international table inclusion

- Add link to Markdown syntax guide in project notes

- Remove commented-out GAM94_RF mortality table code

- Remove deprecated actuarial tables and update tables path in config

- Update error reference documentation for interpolation and mortality method options

- Clean up code structure and remove unused code blocks

- Add .editorconfig for consistent coding styles across files

- Update CI/CD workflow to include macOS and Ubuntu in build matrix

- Update Python version to 3.14 and enable FORCE_JAVASCRIPT_ACTIONS_TO_NODE24

- Enable FORCE_JAVASCRIPT_ACTIONS_TO_NODE24 in CI/CD workflow

- Enable FORCE_JAVASCRIPT_ACTIONS_TO_NODE24 in release-test workflow

- Add FORCE_JAVASCRIPT_ACTIONS_TO_NODE24 environment variable to sync workflow

- Add FORCE_JAVASCRIPT_ACTIONS_TO_NODE24 environment variable to validate-docs workflow

- Enable FORCE_JAVASCRIPT_ACTIONS_TO_NODE24 in validate-wheels workflow

- Enhance CI/CD documentation and streamline module imports for clarity

- Update implementation plan and validate workflows for macOS/Linux support

- Include Windows in the validation workflow OS matrix for comprehensive testing

- Update CI/CD implementation plan with latest phase completions and documentation adjustments

- Update CI/CD implementation plan with completion status and repository renaming

- Update CI/CD implementation plan and licensing document with current status and completed prerequisites

- Add webhook scaffold for Vercel deployment (Fase B)

- Update licensing document with latest revision and webhook scaffold details

- Update project URL in licensing document for Vercel deployment

- Remove redundant requirements.txt (pyproject.toml is used by Vercel)



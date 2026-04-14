## [Unreleased]

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



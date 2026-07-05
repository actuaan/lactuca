## [0.1.0] - 2026-07-02

### ✅ Testing

- **tests**: Relax test_56 golden ULP tolerance for wheel_test


### ✨ Features

- **utils,docs,tests**: Align cashflow utilities across code and AI docs


### 🐛 Bug Fixes

- **tables**: Replace horizontal pl.concat with insert_column

- **ci**: Select wheel ABI matching E2E Python interpreter


### 📚 Documentation

- **legal**: Set EULA and privacy headers to v1.0 for production release

- **legal**: Correct EULA and privacy effective dates to 2026-07-02

- **ai**: Add dates API to AI context pack and agent rules

- **api**: Align first-death and pure-endowment terminology

- Align LICENSE with CC BY carve-out and EULA reference


### 🔧 CI/CD

- **ci**: Avoid duplicate pytest on release push

- **ci**: Disable pre-push pytest by default

- **tests**: Fail pytest when runtime deps lag behind CI floors

- Harden wheel license smoke and align release workflows

- Fix wheel numerical smoke with table interest_rate

- Add linux canary scope and fix wheel pytest layout

- Complete wheel pytest layout and fix polars date deprecation

- Unify wheel E2E across release-test and production

- Copy swiss SOA fixture into isolated wheel pytest root



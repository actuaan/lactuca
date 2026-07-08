# Lactuca — Copilot instructions (end user)

Use when building actuarial scripts with the **Lactuca** Python library (pip install, Python ≥ 3.12).

## API

- `from lactuca import LifeTable, äx, Ax, config, payment_times, tiered_amounts, alb, …`
- Static tables: `LifeTable('TABLE_ID', 'm')`. Generational tables also need `cohort=`.
- OOP and functional APIs are equivalent: `lt.äx(x, n=15)` ≡ `äx(lt, x, n=15)`.
- Interest: **`ir=`** or `table.interest_rate`; never invent `rate=`.

## Pitfalls

- `ex` vs `ex_curtate` (complete vs curtate) vs `ex_continuous` (integer vs fractional age).
- Batch-only: `record_ids`, `on_error`, `benefits=`, `t_output`.
- `äx` ≠ `ax`. **`cashflow_times`** / **`cashflow_amounts`** — `discrete_precision` only;
  `cashflow_times` on immediate paths only; `cashflow_amounts` on due and immediate paths
  (not with `gr=`).
- **`payment_times`**, **`tiered_amounts`** — schedule helpers for irregular / tiered
  cashflows (API: `/api/utils.html`).
- **`alb`** / **`age_last_birthday`** — derive `x` from dates; `config.date_format` for
  `dmy`/`mdy` strings. Dates API: `/api/dates.html`.
- `nEx*` without `gr` or `d`. Joint annuities/insurances accept `m`, `d`, `gr` like single-life products.
- Default `discrete_precision`; batch `return_flows` incompatible with simplified modes.

## Authority

Do not invent API. Docs: https://www.lactuca.io/latest/api/index.html  
Full context file: https://www.lactuca.io/latest/_static/ai/lactuca-ai-core.md

User remains responsible for actuarial correctness (EULA §10.2–10.3).

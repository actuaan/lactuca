# Lactuca — prompt templates

Attach `lactuca-ai-core.md` (and optionally `lactuca-ai-batch.md`) with every prompt.

When a template mentions the package version, use `compatible_with_docs` from the core
file header or `import lactuca; lactuca.__version__`.

---

## 1. Temporary annuity-due (smoke / beginner)

```
Context: attached lactuca-ai-core.md (compatible_with_docs in file header).
Task: Temporary annuity-due, 15 years, male age 50, table PER2020_Ind_1o, cohort=1960, i=3%.
Use äx, discrete_precision. Show imports and a runnable script.
Do not invent parameters not in the context or API reference.
```

---

## 2. Term insurance

```
Context: lactuca-ai-core.md attached.
Task: Net single premium for 20-year term insurance, male 45, PASEM2020_NoRel_1o, i=3%.
Use Ax(n=20). OOP or functional API — your choice, but be consistent.
```

---

## 3. Portfolio batch with benefits

```
Context: lactuca-ai-core.md + lactuca-ai-batch.md attached.
Task: Vector of ages [55, 60, 65], table PER2020_Ind_1o with cohort=1961 (same table
for all policies), per-policy benefits [100, 200, 150], 20-year temporary annuity-due,
m=12, i=3%. Use batch äx with benefits=.
Avoid naive Python loops with cohort setter.
```

---

## 4. Life expectancy at fractional age

```
Context: lactuca-ai-core.md attached.
Task: Complete expectation of life at age 60.5 for PER2020_Ind_1o male, cohort=1960.
Use ex_continuous — not ex. Explain why.
```

---

## 5. Debug ValueError

```
Context: lactuca-ai-core.md attached.
I ran this code and got the traceback below. Fix using only valid Lactuca API.
Do not use record_ids or on_error in scalar mode.

<paste traceback and code>
```

---

## 6. Tiered benefit schedule

```
Context: lactuca-ai-core.md attached.
Task: 20-year annual annuity-due, male 50, PER2020_Ind_1o, cohort=1960, i=3%.
Benefit 12,000 for years 1–10 and 8,000 thereafter.
Use payment_times + tiered_amounts, then äx with cashflow_amounts (discrete_precision).
Do not use np.where for tier assignment — use tiered_amounts.
```

---

## 7. Entry age from birth and valuation dates

```
Context: lactuca-ai-core.md attached.
Task: Price a 20-year temporary annuity-due for a male born 1960-05-15, valuation date
2024-06-01, table PER2020_Ind_1o, cohort=1960, i=3%.
Use alb (or age_last_birthday) to obtain entry age x, then äx. Set config.date_format if
needed. Do not hand-roll year differences or confuse age_exact with ex.
```

# Date Utilities Guide

The `lactuca.dates` module provides vectorized date-manipulation and actuarial
age-calculation functions. Date-oriented functions accept the broad range of input
types listed in the `DateLike` section below. The module covers seven function groups:
date construction and manipulation, component extraction, formatting, duration
calculation, actuarial age, anniversary generation, and calendar utilities.

## Accepted date formats (`DateLike`)

Date-oriented input functions in `lactuca.dates` accept `DateLike` inputs. In the API reference and
error messages, `DateLike` refers to the set of accepted date input forms.

String and numeric parsing rules depend on `config.date_format` (see
{ref}`dates-config`):

| Format | Example |
|--------|---------|
| `datetime.date` | `date(2024, 1, 15)` |
| `datetime.datetime` | `datetime(2024, 1, 15, 0, 0)` |
| ISO string | `"2024-01-15"` |
| European date string | `"15/01/2024"` (`DD/MM/YYYY`, when `date_format="dmy"`) |
| US date string | `"01/15/2024"` (`MM/DD/YYYY`, when `date_format="mdy"`) |
| Integer YYYYMMDD | `20240115` |
| pandas `Timestamp` | `pd.Timestamp("2024-01-15")` |
| Polars `Date` | Polars date scalar |
| NumPy `datetime64` | `np.datetime64("2024-01-15")` |

Accepted string/number forms by configuration:

- `config.date_format="ymd"`: accepts `YYYY-MM-DD` and `YYYY/MM/DD`
- `config.date_format="dmy"`: accepts `DD/MM/YYYY` and `DD-MM-YYYY`
- `config.date_format="mdy"`: accepts `MM/DD/YYYY` and `MM-DD-YYYY`
- `config.date_format="ymd_int"`: accepts 8-digit integers / numeric strings (`YYYYMMDD`)

Sequence inputs (Python `list`, NumPy array, etc.) are accepted wherever documented.
Age functions return `numpy.ndarray` of `float64`. Duration functions return
`numpy.ndarray` of `int32` for days/months and `float64` for years. Date
construction functions (`make_date`, `end_of_month`, `add_duration`,
`anniversary_dates`) return a `FormatDates` list.

When multiple sequence arguments are provided, lengths must be compatible for
broadcasting (equal lengths, or length 1 to be broadcast). Incompatible lengths
raise `ValueError`.

---

## Date construction and manipulation

### `make_date(year, month, day)` → `FormatDates`

Constructs one or more dates from integer year, month, and day components.  Returns a
`FormatDates` object — a list of `datetime.date` values with a `.format()` method for
string conversion.  Any of the three arguments may be a sequence; scalar arguments are
broadcast to match the longest sequence.

```python
from lactuca.dates import make_date

d = make_date(2024, 3, 15)
print(d)           # FormatDates([datetime.date(2024, 3, 15)])
print(d.format())  # '2024-03-15'

# Vectorised: build one date per month, same year and day
monthly = make_date(2024, [1, 2, 3, 4, 5, 6], 1)
print(monthly.format())
# ['2024-01-01', '2024-02-01', '2024-03-01', '2024-04-01', '2024-05-01', '2024-06-01']
```

### `end_of_month(input_date)` → `FormatDates`

Returns the last calendar day of the month containing `input_date`.

```python
from lactuca.dates import end_of_month

print(end_of_month("2024-02-10"))  # FormatDates([datetime.date(2024, 2, 29)])  (2024 is leap)
print(end_of_month("2023-02-10"))  # FormatDates([datetime.date(2023, 2, 28)])
```

### `add_duration(start_date, *, years=0, months=0, days=0)` → `FormatDates`

Adds an offset of years, months, and/or days to a date.  When the resulting
day-of-month does not exist in the target month, it is clamped to the last valid day
(e.g. January 31 + 1 month → February 28/29).

```python
from lactuca.dates import add_duration

# End-of-month clamping
print(add_duration("2024-01-31", months=1))  # FormatDates([datetime.date(2024, 2, 29)])

# Add one year
print(add_duration("2023-06-15", years=1))   # FormatDates([datetime.date(2024, 6, 15)])

# Combine years, months and days in one call
print(add_duration("2024-01-15", years=1, months=2, days=10))  # FormatDates([datetime.date(2025, 3, 25)])
```

---

## Date components

### `year`, `month`, `day`

Extract a single component from a date. Sequence inputs return a NumPy array
(`int32`).

```python
from lactuca.dates import year, month, day

print(year("2024-07-15"))   # 2024
print(month("2024-07-15"))  # 7
print(day("2024-07-15"))    # 15
```

### `quarter(input_date, *, prefix=None, suffix=None)`

Returns the calendar quarter (1–4).  Optional `prefix` and `suffix` strings are
prepended and appended to the result.

```python
from lactuca.dates import quarter

print(quarter("2024-07-15"))                               # 3
print(quarter("2024-07-15", prefix="Q"))                  # 'Q3'    (Anglo-Saxon)
print(quarter("2024-07-15", suffix="T"))                  # '3T'    (Spanish)
print(quarter("2024-07-15", prefix="Q", suffix="/2024"))  # 'Q3/2024'

# Sequence input
print(quarter(["2024-01-15", "2024-07-15", "2024-12-31"]))
# [1, 3, 4]
```

---

## Date formatting

### `FormatDates` and `.format()`

Functions such as `make_date`, `end_of_month`, `add_duration`, `next_anniversary`, and
`anniversary_dates` return a `FormatDates` object — a `list` subclass of
`datetime.date` values. Call `.format(date_format=None)` to convert to the
configured output representation.
When `date_format=None` the global `config.date_format` setting is used.

`.format()` returns a **scalar** when the wrapper contains a single date, and a
**list** when it contains multiple dates. Values are strings for
`"ymd"`, `"dmy"`, and `"mdy"`, and integers for `"ymd_int"`.

```python
from lactuca.dates import make_date, anniversary_dates
from datetime import date

# Single date — scalar string
dates = make_date(2024, 7, 15)
print(dates.format())        # '2024-07-15'
print(dates.format("dmy"))   # '15/07/2024'

# Multiple dates — list of strings
anns = anniversary_dates(date(2024, 1, 1), date(2025, 1, 1), m=4)
print(anns.format())
# ['2024-01-01', '2024-04-01', '2024-07-01', '2024-10-01']
```

### `format_date(input_date, date_format=None)`

Formats any `DateLike` input using the specified output format.
Input parsing follows the global `config.date_format` setting. When
`date_format="ymd_int"`, the result is an **integer** in `YYYYMMDD` form; other
formats return strings.

```python
from lactuca.dates import format_date

print(format_date("2024-07-15", date_format="dmy"))      # '15/07/2024'
print(format_date("2024-07-15", date_format="ymd"))      # '2024-07-15'
print(format_date("2024-07-15", date_format="ymd_int"))  # 20240715  (integer)
```

---

## Duration calculations

### `days_between(date1, date2)`

Returns the **integer** number of days ($\text{date}_2 - \text{date}_1$).

```python
from datetime import date
from lactuca.dates import days_between

print(days_between(date(2024, 1, 1), date(2024, 12, 31)))  # 365
```

### `months_between(date1, date2)`

Returns the number of **complete** calendar months elapsed.  A month is complete when
the destination day-of-month is ≥ the start day-of-month.

```python
from lactuca.dates import months_between

print(months_between("2024-01-15", "2024-06-20"))  # 5
```

### `years_between(date1, date2, *, method='act_act')`

Returns fractional years between two dates.

| `method` | Convention | Notes |
|----------|-----------|-------|
| `'act_act'` | Actual/Actual ISDA | Year-by-year leap-year accumulation; highest precision |
| `'exact'` | days / 365.25 | Faster; maximum error < 0.003 years |

```python
from lactuca.dates import years_between

# Aligned 4-year span — both methods agree (1461 days / 365.25 = 4.0 exactly)
print(years_between("2020-01-01", "2024-01-01", method="act_act"))  # 4.0
print(years_between("2020-01-01", "2024-01-01", method="exact"))    # 4.0

# 3 non-leap years — methods diverge (1095 days / 365.25 < 3.0)
print(years_between("2021-01-01", "2024-01-01", method="act_act"))  # 3.0
print(years_between("2021-01-01", "2024-01-01", method="exact"))    # 2.9979466119096503
```

### `time_diff(date1, date2, *, unit='years', method=None)`

Unified entry point for all duration calculations.  `unit` may be `'days'`, `'months'`,
or `'years'`.  For `unit='years'`, `method` is passed to `years_between` (defaults to
`'act_act'`).

```python
from lactuca.dates import time_diff

print(time_diff("2024-01-01", "2024-12-31", unit="days"))    # 365
print(time_diff("2024-01-15", "2024-06-20", unit="months"))  # 5
print(time_diff("2020-01-01", "2024-01-01", unit="years"))   # 4.0
```

---

## Actuarial age

All age functions accept scalar dates or sequences.  A scalar input returns a Python
`float`; a sequence returns a `numpy.ndarray` of `float64`.

### `age_exact(birth_date, valuation_date, *, day_count='act_act')`

Exact fractional age in years, with no rounding.  Wraps
`act_age(birth_date, valuation_date, m=365, method='exact', day_count=day_count)`.
Uses Actual/Actual ISDA by default; pass `day_count='exact'` for the faster
days / 365.25 approximation (maximum error ≈ 0.003 years).

```python
from lactuca.dates import age_exact

# On exact birthday — integer result
print(age_exact("1990-01-01", "2024-01-01"))                      # 34.0

# Mid-year valuation — fractional result (Actual/Actual ISDA)
print(age_exact("1990-01-01", "2024-07-01"))                      # 34.49726775956284

# Fast approximation (days / 365.25)
print(age_exact("1990-01-01", "2024-07-01", day_count="exact"))   # 34.496919917864474
```

### Rounded age conventions

The three convenience functions apply standard actuarial rounding at integer birthday
boundaries:

| Function | Notation | Formula | Description |
|----------|----------|---------|-------------|
| `age_last_birthday` | ALB, $[x]$ | $\lfloor x_{\text{exact}} \rfloor$ | Rounded **down** to last integer birthday |
| `age_nearest_birthday` | ANB | $\operatorname{round}(x_{\text{exact}})$ | Rounded to **nearest** integer birthday |
| `age_next_birthday` | ANEXT | $\lceil x_{\text{exact}} \rceil$ | Rounded **up** to next integer birthday |

All three share the same signature:
`(birth_date, valuation_date, *, day_count='act_act')`.

Short-form aliases `alb`, `anb`, and `anextb` are exported at the top-level
`lactuca` namespace for interactive use and scripting.  The name `anextb`
(not `anext`) is used to avoid shadowing `builtins.anext` (Python 3.10+).

```python
import lactuca as lc

birth, val = "1990-06-15", "2024-03-01"  # exact age ≈ 33.7

print(lc.alb(birth, val))     # 33.0   (Age Last Birthday)
print(lc.anb(birth, val))     # 34.0   (Age Nearest Birthday)
print(lc.anextb(birth, val))  # 34.0   (Age Next Birthday)
```

### `act_age(birth_date, valuation_date, *, m=365, method='exact', day_count='act_act')`

General actuarial age function with selectable rounding frequency `m` and method.
Calling `act_age(birth, val)` with the defaults is identical to `age_exact(birth, val)`.

| `method` | Description |
|----------|-------------|
| `'exact'` | No rounding — exact fractional age (same as `age_exact`). Requires `m=365`. |
| `'last'` | Rounds **down** to the nearest $1/m$ year (ALB when $m=1$) |
| `'nearest'` | Rounds to the **nearest** $1/m$ year (ANB when $m=1$) |
| `'next'` | Rounds **up** to the nearest $1/m$ year (ANEXT when $m=1$) |

To compute age rounded down to the nearest completed month ($m=12$, `method='last'`):

```python
from datetime import date
from lactuca.dates import act_age

# Monthly ALB: age rounded down to the nearest 1/12 year
print(act_age("1989-07-01", "2024-01-01", m=12, method="last"))  # 34.5

# Vectorised: one call for a portfolio of insured persons
births = [date(1960, 1, 1), date(1975, 6, 15), date(1990, 12, 1)]
print(act_age(births, date(2024, 1, 1), m=1, method="last"))
# array([64., 48., 33.])
```

:::{note}
`method='exact'` enforces `m=365`.  Passing any other `m` value raises `ValueError`.
:::

---

## Anniversary generation

### `next_anniversary(birth_date, ref_date)`

Returns the next occurrence of the birth month/day **strictly after** `ref_date`.
If `ref_date` falls exactly on an anniversary, the following year's anniversary
is returned.

```python
from lactuca.dates import next_anniversary

result = next_anniversary("1990-06-15", "2024-03-01")
print(result)           # FormatDates([datetime.date(2024, 6, 15)])
print(result.format())  # '2024-06-15'
```

### `anniversary_dates(start_date, end_date, *, m=1, selected_periods=None)` → `FormatDates`

Returns all $m$-thly anniversary dates from `start_date` (inclusive) to `end_date`
(exclusive).  `m` must be one of `{1, 2, 3, 4, 6, 12, 24, 26, 52, 365}`.
`m=24` uses a 15-day interval (360-day commercial convention).
`m=26` uses a 14-day interval (biweekly, 364-day convention).
`m=14` is **not supported** and raises `ValueError`; model "14 pagas" as `m=12` plus
two extraordinary cash flows instead.

```python
from datetime import date
from lactuca.dates import anniversary_dates

# Annual payment dates over two years
anns = anniversary_dates(date(2024, 1, 1), date(2026, 1, 1), m=1)
print(anns.format())
# ['2024-01-01', '2025-01-01']

# Quarterly payment grid for 2024
quarterly = anniversary_dates(date(2024, 1, 1), date(2025, 1, 1), m=4)
print(quarterly.format())
# ['2024-01-01', '2024-04-01', '2024-07-01', '2024-10-01']
```

Use `selected_periods` to restrict to a subset of period indices within the range.

---

## Calendar utilities

### `is_leap_year(year)`

Returns `True` if `year` is a Gregorian leap year.

```python
from lactuca.dates import is_leap_year

print(is_leap_year(2024))  # True
print(is_leap_year(1900))  # False  (divisible by 100 but not by 400)
```

### `days_in_year(year)` and `days_in_month(year, month)`

```python
from lactuca.dates import days_in_year, days_in_month

print(days_in_year(2024))      # 366
print(days_in_year(2023))      # 365
print(days_in_month(2024, 2))  # 29  (leap year)
print(days_in_month(2023, 2))  # 28
```

---

## Vectorized usage

All date functions broadcast over Python lists, NumPy arrays, and other sequences:

```python
from datetime import date
from lactuca.dates import age_last_birthday, years_between

# Array of births, single valuation date
births = [date(1960, 3, 15), date(1975, 11, 1), date(1990, 6, 30)]
ages = age_last_birthday(births, date(2024, 1, 1))
print(ages)
# array([63., 48., 33.])  (numpy.ndarray of float64)

# Array of start dates paired with a common end date
starts = ["2020-01-01", "2021-01-01"]
ends   = ["2024-01-01", "2024-01-01"]
print(years_between(starts, ends))
# array([4., 3.])
```

---

(dates-config)=
## Configuration

| Setting | Default | Allowed values | Description |
|---------|---------|---------------|-------------|
| `config.date_format` | `"ymd"` | `"ymd"`, `"dmy"`, `"mdy"`, `"ymd_int"` | Controls parsing of ambiguous date inputs and default date output format (`"ymd"` → `YYYY-MM-DD`, `"dmy"` → `DD/MM/YYYY`, `"mdy"` → `MM/DD/YYYY`, `"ymd_int"` → integer `YYYYMMDD`) |


```python
from lactuca import config
from lactuca.dates import format_date

config.date_format = "dmy"
print(format_date("2024-07-15"))  # '15/07/2024'
config.reset()                    # restore defaults
```

---

## See also

- {doc}`notation_glossary` — age notation: $x$, $[x]$, $x_{\text{ANB}}$
- {doc}`interest_rates_guide` — how `days_per_year` affects interest factors
- {doc}`decimals_rounding` — global decimal precision settings

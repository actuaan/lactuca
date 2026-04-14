# Date Utilities

High-performance date manipulation and actuarial age/duration functions following
Spanish and international conventions.  Key capabilities:

- **Actuarial age** — Age Last Birthday (ALB), Age Nearest Birthday (ANB), Age Next
  Birthday (ANEXT), and exact fractional age with daily resolution.
- **Duration calculations** — days, complete calendar months, and fractional years
  (Actual/Actual ISDA or days/365.25 approximation).
- **Anniversary dates** — generation of payment or projection date grids at any
  frequency ($m$ = 1, 2, 3, 4, 6, 12, 24, 26, 52, 365; `m=14` raises ``ValueError`` —
  use `m=12` with separate extraordinary cash flows for the Spanish "14 pagas" scheme).
- **Date construction and arithmetic** — end-of-month dates, durations, formatting,
  and robust parsing of integers (`YYYYMMDD`), ISO strings, European and US slash formats,
  and third-party types (``datetime.date``, ``pandas.Timestamp``, ``numpy.datetime64``, Polars).
- **Calendar utilities** — leap year detection, days in month, days in year.

The active ``date_format`` setting in {class}`~lactuca.Config` controls which input
representations are accepted and how output is formatted.

```{seealso}
{doc}`../user_guide/dates_guide` — Full guide to date parsing, format configuration,
and actuarial age conventions.
```

## Actuarial age

```{eval-rst}
.. autofunction:: lactuca.age_last_birthday
.. autofunction:: lactuca.age_nearest_birthday
.. autofunction:: lactuca.age_next_birthday
.. autofunction:: lactuca.age_exact
.. autofunction:: lactuca.act_age
```

### Compact aliases

Short-form aliases are exported directly in the top-level `lactuca` namespace:

| Alias | Equivalent | Convention |
|-------|-----------|------------|
| `lactuca.alb` | `age_last_birthday` | ALB — floor to last integer year |
| `lactuca.anb` | `age_nearest_birthday` | ANB — round to nearest integer year |
| `lactuca.anextb` | `age_next_birthday` | ANEXTB — ceiling to next integer year |

The name `anextb` (not `anext`) is used to avoid shadowing `builtins.anext`
(Python 3.10+, used for async iteration).

## Duration calculations

```{eval-rst}
.. autofunction:: lactuca.days_between
.. autofunction:: lactuca.months_between
.. autofunction:: lactuca.years_between
.. autofunction:: lactuca.time_diff
```

## Anniversary dates

```{eval-rst}
.. autofunction:: lactuca.anniversary_dates
.. autofunction:: lactuca.next_anniversary
```

## Date construction and arithmetic

### Date input types

```{eval-rst}
.. py:data:: lactuca.dates.DateLike
   :type: Union[int, str, datetime.date, datetime.datetime, pandas.Timestamp, numpy.datetime64, polars.Date, polars.Datetime]

   Union type alias for all date-like inputs accepted by the public API.

   Supported types:

   - :class:`int` — 8-digit integer ``YYYYMMDD`` (e.g. ``20240115``)
   - :class:`str` — ISO ``YYYY-MM-DD``, European slash ``DD/MM/YYYY``,
     European dash ``DD-MM-YYYY``, or YMD slash ``YYYY/MM/DD``
   - :class:`datetime.date` or :class:`datetime.datetime`
   - :class:`pandas.Timestamp`
   - :class:`numpy.datetime64`
   - Polars ``Date`` / ``Datetime`` scalars

   The active :attr:`~lactuca.Config.date_format` setting controls which
   string representations are accepted (e.g. ISO-only, European formats, etc.).
```

```{eval-rst}
.. autofunction:: lactuca.make_date
.. autofunction:: lactuca.add_duration
.. autofunction:: lactuca.end_of_month
.. autofunction:: lactuca.format_date
.. autoclass:: lactuca.FormatDates
   :members:
```

## Calendar utilities

```{eval-rst}
.. autofunction:: lactuca.is_leap_year
.. autofunction:: lactuca.days_in_year
.. autofunction:: lactuca.days_in_month
```

## Date component extraction

```{eval-rst}
.. autofunction:: lactuca.year
.. autofunction:: lactuca.month
.. autofunction:: lactuca.day
.. autofunction:: lactuca.quarter
```

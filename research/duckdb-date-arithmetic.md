# DuckDB Date Arithmetic

DuckDB supports a variety of operations for date and timestamp arithmetic.

## Date and Interval Operators

You can perform addition and subtraction on dates and timestamps using integers (representing days) or `INTERVAL` types.

| Operator | Description | Example | Result |
|---|---|---|---|
| `+` | addition of days (integers) | `DATE '1992-03-22' + 5` | `1992-03-27` |
| `+` | addition of an `INTERVAL` | `DATE '1992-03-22' + INTERVAL 5 DAY` | `1992-03-27 00:00:00` |
| `-` | subtraction of `DATE`s | `DATE '1992-03-27' - DATE '1992-03-22'` | `5` (days) |
| `-` | subtraction of `TIMESTAMP`s | `TIMESTAMP '1992-03-27 10:00:00' - TIMESTAMP '1992-03-22 08:30:00'` | `5 days 01:30:00` (`INTERVAL`) |
| `-` | subtraction of an `INTERVAL` | `DATE '1992-03-27' - INTERVAL 5 DAY` | `1992-03-22 00:00:00` |

## Date Functions

DuckDB provides a rich set of functions for date manipulation.

- `date_add(date, interval)`: Adds an interval to a date.
- `date_diff(part, startdate, enddate)`: Returns the number of `part` boundaries between two dates.
- `date_trunc(part, date)`: Truncates a date to a specified precision.
- `extract(part from date)`: Extracts a subfield from a date (e.g., 'year', 'month', 'day').
- `strftime(date, format)`: Formats a date into a string.

## Extracting Date Parts

The `date_part` or `extract` function can be used to get specific parts of a date or timestamp, such as the year, month, day, hour, etc.

```sql
SELECT date_part('year', DATE '1992-09-20'); -- Returns 1992
SELECT extract('hour' FROM TIMESTAMP '2023-10-26 10:30:00'); -- Returns 10
```

This is useful for getting results as integers.

## Extracting from Intervals

You can also extract parts from an `INTERVAL` value.

```sql
SELECT EXTRACT('hour' FROM INTERVAL 5 days 01:30:00); -- Returns 1
SELECT EXTRACT('minute' FROM INTERVAL 5 days 01:30:00); -- Returns 30
```

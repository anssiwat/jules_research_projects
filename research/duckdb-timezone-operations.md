# DuckDB Timezone Operations

DuckDB provides robust support for timezone operations, primarily through the `TIMESTAMPTZ` data type and the ICU extension.

## `TIMESTAMPTZ` Data Type

The `TIMESTAMPTZ` or `TIMESTAMP WITH TIME ZONE` data type stores timestamps as the number of microseconds since the Unix epoch (1970-01-01 00:00:00+00). It does not store the timezone itself, but uses the session's timezone setting for display and calculations.

## Setting the Timezone

To work with timezones, you first need to load the ICU extension and then set the desired timezone.

```sql
INSTALL icu;
LOAD icu;
SET TimeZone = 'America/Los_Angeles';
```

A list of available timezones can be retrieved using:
```sql
SELECT name, abbrev FROM pg_timezone_names() ORDER BY name;
```

## Timezone Conversion

The `timezone()` function can be used to convert timestamps between timezones.

```sql
SELECT timezone('America/Denver', TIMESTAMP '2001-02-16 20:38:40');
```

The standard SQL `AT TIME ZONE` operator is also supported.

```sql
SELECT TIMESTAMP '2001-02-16 20:38:40' AT TIME ZONE 'America/Denver';
```

## Daylight Saving Time (DST)

`TIMESTAMPTZ` automatically handles Daylight Saving Time. When performing arithmetic that crosses a DST boundary, the results will reflect the change.

For example, in the US, DST typically starts on the second Sunday in March. Let's look at an example in the `America/New_York` timezone.

```sql
SET TimeZone = 'America/New_York';
-- In 2023, DST started on March 12th.
-- Adding 1 day to a timestamp before the change correctly handles the time shift.
SELECT TIMESTAMPTZ '2023-03-11 01:30:00' + INTERVAL 1 DAY;
-- Result: 2023-03-12 02:30:00-04
-- Notice the time is 02:30, not 01:30, and the offset has changed from -05 to -04.
```

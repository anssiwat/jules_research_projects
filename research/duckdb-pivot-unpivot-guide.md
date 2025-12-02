# DuckDB PIVOT and UNPIVOT Guide

## Table of Contents

1.  [Introduction](#introduction)
    *   [What is PIVOT?](#what-is-pivot)
2.  [Simplified PIVOT Syntax](#simplified-pivot-syntax)
    *   [Example Data](#example-data)
    *   [PIVOT ON and USING](#pivot-on-and-using)
    *   [PIVOT ON, USING, and GROUP BY](#pivot-on-using-and-group-by)
    *   [IN Filter for ON Clause](#in-filter-for-on-clause)
    *   [Multiple Expressions per Clause](#multiple-expressions-per-clause)
    *   [Using PIVOT within a SELECT Statement](#using-pivot-within-a-select-statement)
    *   [Multiple PIVOT Statements](#multiple-pivot-statements)
3.  [SQL Standard PIVOT Syntax](#sql-standard-pivot-syntax)
    *   [Examples](#examples)
4.  [Advanced PIVOT Examples](#advanced-pivot-examples)
    *   [Pivoting with Multiple Aggregates and a Total Column](#pivoting-with-multiple-aggregates-and-a-total-column)
5.  [PIVOT Internals](#pivot-internals)
6.  [UNPIVOT Statement](#unpivot-statement)
    *   [UNPIVOT Internals](#unpivot-internals)
7.  [Limitations](#limitations)

---

## 1. Introduction

### What is PIVOT?

The `PIVOT` statement is a powerful tool for transforming data from a long format to a wide format. It allows you to rotate rows into columns, effectively turning unique values in a column into new columns in your output. The values in these new columns are calculated using an aggregate function on the subset of rows that match each distinct value.

DuckDB provides two syntaxes for the `PIVOT` statement: a simplified, more intuitive syntax, and the SQL Standard syntax. The simplified syntax is particularly useful for dynamic pivoting, where the new column names are not known in advance.

The inverse of `PIVOT` is `UNPIVOT`, which transforms data from a wide format back to a long format.

## 2. Simplified PIVOT Syntax

The simplified `PIVOT` syntax is designed to be user-friendly and can be summarized using spreadsheet pivot table terminology:

```sql
PIVOT dataset
ON columns
USING values
GROUP BY rows
ORDER BY columns_with_order_directions
LIMIT number_of_rows;
```

The `ON`, `USING`, and `GROUP BY` clauses are all optional, but at least one must be provided.

### Example Data

All examples in this guide will use the following `cities` table:

```sql
CREATE TABLE cities (
    country VARCHAR, name VARCHAR, year INTEGER, population INTEGER
);
INSERT INTO cities VALUES
    ('NL', 'Amsterdam', 2000, 1005),
    ('NL', 'Amsterdam', 2010, 1065),
    ('NL', 'Amsterdam', 2020, 1158),
    ('US', 'Seattle', 2000, 564),
    ('US', 'Seattle', 2010, 608),
    ('US', 'Seattle', 2020, 738),
    ('US', 'New York City', 2000, 8015),
    ('US', 'New York City', 2010, 8175),
    ('US', 'New York City', 2020, 8772);
```

A `SELECT *` from the `cities` table returns:

```
┌─────────┬───────────────┬───────┬────────────┐
│ country │     name      │ year  │ population │
│ varchar │    varchar    │ int32 │   int32    │
├─────────┼───────────────┼───────┼────────────┤
│ NL      │ Amsterdam     │  2000 │       1005 │
│ NL      │ Amsterdam     │  2010 │       1065 │
│ NL      │ Amsterdam     │  2020 │       1158 │
│ US      │ Seattle       │  2000 │        564 │
│ US      │ Seattle       │  2010 │        608 │
│ US      │ Seattle       │  2020 │        738 │
│ US      │ New York City │  2000 │       8015 │
│ US      │ New York City │  2010 │       8175 │
│ US      │ New York City │  2020 │       8772 │
└─────────┴───────────────┴───────┴────────────┘
```

### PIVOT ON and USING

The `ON` clause specifies which column's values will become the new column headers. The `USING` clause specifies the aggregation to be performed on the values for the new columns.

```sql
PIVOT cities
ON year
USING sum(population);
```

This query will create a new column for each distinct year, and the values will be the sum of the population for that year.

```
┌─────────┬───────────────┬──────┬──────┬──────┐
│ country │     name      │ 2000 │ 2010 │ 2020 │
│ varchar │    varchar    │ int  │ int  │ int  │
├─────────┼───────────────┼──────┼──────┼──────┤
│ NL      │ Amsterdam     │ 1005 │ 1065 │ 1158 │
│ US      │ Seattle       │ 564  │ 608  │ 738  │
│ US      │ New York City │ 8015 │ 8175 │ 8772 │
└─────────┴───────────────┴──────┴──────┴──────┘
```

### PIVOT ON, USING, and GROUP BY

By default, all columns not specified in the `ON` or `USING` clauses are used for grouping. To specify the grouping columns explicitly, use the `GROUP BY` clause.

```sql
PIVOT cities
ON year
USING sum(population)
GROUP BY country;
```

This will aggregate the data at the country level, removing the `name` column from the output.

```
┌─────────┬──────┬──────┬──────┐
│ country │ 2000 │ 2010 │ 2020 │
│ varchar │ int  │ int  │ int  │
├─────────┼──────┼──────┼──────┤
│ NL      │ 1005 │ 1065 │ 1158 │
│ US      │ 8579 │ 8783 │ 9510 │
└─────────┴──────┴──────┴──────┘
```

### IN Filter for ON Clause

You can filter the values that become new columns by using an `IN` clause with the `ON` column.

```sql
PIVOT cities
ON year IN (2000, 2010)
USING sum(population)
GROUP BY country;
```

This will only create columns for the years 2000 and 2010.

```
┌─────────┬──────┬──────┐
│ country │ 2000 │ 2010 │
│ varchar │ int  │ int  │
├─────────┼──────┼──────┤
│ NL      │ 1005 │ 1065 │
│ US      │ 8579 │ 8783 │
└─────────┴──────┴──────┘
```

### Multiple Expressions per Clause

You can use multiple columns in the `ON` and `GROUP BY` clauses, and multiple aggregate expressions in the `USING` clause.

```sql
PIVOT cities
ON year
USING sum(population) AS total, max(population) AS max
GROUP BY country;
```

This will create columns for both the sum and max of the population for each year.

```
┌─────────┬──────────┬────────┬──────────┬────────┬──────────┬────────┐
│ country │ 2000_total │ 2000_max │ 2010_total │ 2010_max │ 2020_total │ 2020_max │
│ varchar │   int    │  int   │   int    │  int   │   int    │  int   │
├─────────┼──────────┼────────┼──────────┼────────┼──────────┼────────┤
│ US      │   8579   │  8015  │   8783   │  8175  │   9510   │  8772  │
│ NL      │   1005   │  1005  │   1065   │  1065  │   1158   │  1158  │
└─────────┴──────────┴────────┴──────────┴────────┴──────────┴────────┘
```

### Using PIVOT within a SELECT Statement

The `PIVOT` statement can be used within a `SELECT` statement as a Common Table Expression (CTE) or a subquery.

```sql
WITH pivot_alias AS (
    PIVOT cities
    ON year
    USING sum(population)
    GROUP BY country
)
SELECT * FROM pivot_alias;
```

### Multiple PIVOT Statements

You can use multiple `PIVOT` statements in a single query, for example, by joining them.

```sql
SELECT *
FROM (PIVOT cities ON year USING sum(population) GROUP BY country) year_pivot
JOIN (PIVOT cities ON name USING sum(population) GROUP BY country) name_pivot
USING (country);
```

This will create a wider pivot with columns for both years and city names.

## 3. SQL Standard PIVOT Syntax

The SQL Standard PIVOT syntax is more verbose than the simplified syntax, but it offers more control over the output. The general form is:

```sql
SELECT *
FROM dataset
PIVOT (
    values
    FOR
        column_1 IN (in_list)
        column_2 IN (in_list)
        ...
    GROUP BY rows
);
```

### Examples

Here is an example of the SQL Standard PIVOT syntax:

```sql
SELECT *
FROM cities
PIVOT (
    sum(population)
    FOR
        year IN (2000, 2010, 2020)
    GROUP BY country
);
```

This produces the same result as the simplified syntax example with the `GROUP BY` clause.

## 4. Advanced PIVOT Examples

### Pivoting with Multiple Aggregates and a Total Column

In this example, we will calculate the number of players in each conference for each year, and also include a total player count for each conference.

First, let's create the tables and data for this example:

```sql
CREATE TABLE teams (
    school_name VARCHAR,
    conference VARCHAR
);

CREATE TABLE players (
    school_name VARCHAR,
    year VARCHAR
);

INSERT INTO teams VALUES
    ('Stanford', 'Pac-12'),
    ('Cal', 'Pac-12'),
    ('UCLA', 'Pac-12'),
    ('USC', 'Pac-12'),
    ('Oregon', 'Pac-12'),
    ('Washington', 'Pac-12'),
    ('Washington State', 'Pac-12'),
    ('Oregon State', 'Pac-12'),
    ('Colorado', 'Pac-12'),
    ('Utah', 'Pac-12'),
    ('Arizona', 'Pac-12'),
    ('Arizona State', 'Pac-12'),
    ('Alabama', 'SEC'),
    ('LSU', 'SEC'),
    ('Texas A&M', 'SEC'),
    ('Auburn', 'SEC');

INSERT INTO players VALUES
    ('Stanford', 'FR'),
    ('Stanford', 'SO'),
    ('Stanford', 'JR'),
    ('Stanford', 'SR'),
    ('Cal', 'FR'),
    ('Cal', 'SO'),
    ('UCLA', 'FR'),
    ('UCLA', 'SO'),
    ('Alabama', 'FR'),
    ('Alabama', 'SO'),
    ('Alabama', 'JR'),
    ('Alabama', 'SR');
```

Now, we can use the `PIVOT` statement to create the desired output. We will also add a `total_players` column.

```sql
WITH pivoted_data AS (
    PIVOT (
        SELECT teams.conference, players.year
        FROM players
        JOIN teams ON teams.school_name = players.school_name
    )
    ON year
    USING count(*)
    GROUP BY conference
)
SELECT
    conference,
    coalesce("FR", 0) + coalesce("SO", 0) + coalesce("JR", 0) + coalesce("SR", 0) AS total_players,
    "FR", "SO", "JR", "SR"
FROM pivoted_data
ORDER BY total_players DESC;
```

This will produce the following output:

```
┌────────────┬───────────────┬──────┬──────┬──────┬──────┐
│ conference │ total_players │  FR  │  SO  │  JR  │  SR  │
│  varchar   │    int128     │ int8 │ int8 │ int8 │ int8 │
├────────────┼───────────────┼──────┼──────┼──────┼──────┤
│ Pac-12     │             8 │    3 │    3 │    1 │    1 │
│ SEC        │             4 │    1 │    1 │    1 │    1 │
└────────────┴───────────────┴──────┴──────┴──────┴──────┘
```

## 5. PIVOT Internals

The `PIVOT` operation in DuckDB is implemented as a combination of SQL query re-writing and a dedicated `PhysicalPivot` operator. When the `IN` clause is not used, DuckDB first determines the distinct values that will become the new column names by creating a temporary `ENUM` type. The query is then rewritten to use this `ENUM` in the `IN` clause.

The final step involves aggregating the data into lists, which are then transformed by the `PhysicalPivot` operator into the final pivoted table.

## 6. UNPIVOT Statement

The `UNPIVOT` statement is the inverse of `PIVOT`. It transforms columns into rows.

Here is an example of a table with monthly sales data that is in a wide format:

```sql
CREATE TABLE monthly_sales (
    empid INTEGER,
    dept VARCHAR,
    jan INTEGER,
    feb INTEGER,
    mar INTEGER,
    apr INTEGER,
    may INTEGER,
    jun INTEGER
);

INSERT INTO monthly_sales VALUES
    (1, 'electronics', 1, 2, 3, 4, 5, 6),
    (2, 'clothes', 10, 20, 30, 40, 50, 60),
    (3, 'cars', 100, 200, 300, 400, 500, 600);
```

We can use `UNPIVOT` to transform this data into a long format:

```sql
UNPIVOT monthly_sales
ON jan, feb, mar, apr, may, jun
INTO
    NAME month
    VALUE sales;
```

This will produce the following output:

```
┌───────┬─────────────┬─────────┬───────┐
│ empid │    dept     │  month  │ sales │
│ int32 │   varchar   │ varchar │ int32 │
├───────┼─────────────┼─────────┼───────┤
│     1 │ electronics │ jan     │     1 │
│     1 │ electronics │ feb     │     2 │
│     1 │ electronics │ mar     │     3 │
│   ... │ ...         │ ...     │   ... │
└───────┴─────────────┴─────────┴───────┘
```

### UNPIVOT Internals

The `UNPIVOT` operation is implemented as a SQL query rewrite. It uses the `unnest` function to expand the list of column names and values into separate rows.

## 7. Limitations

The `PIVOT` statement in DuckDB currently has a limitation where expressions are not allowed in the `USING` clause. For example, the following query will fail:

```sql
PIVOT cities
ON year
USING sum(population) * 1000;
```

The workaround is to perform the `PIVOT` operation first and then apply the expression to the resulting columns using the `COLUMNS` expression:

```sql
SELECT country, name, 1000 * COLUMNS(* EXCLUDE (country, name))
FROM (
    PIVOT cities
    ON year
    USING sum(population)
);
```

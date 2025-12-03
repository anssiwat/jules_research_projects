# DuckDB ALTER TABLE Statement Guide

## Table of Contents

1.  [Introduction](#introduction)
2.  [Creating the Initial Table](#creating-the-initial-table)
3.  [RENAME TABLE](#rename-table)
4.  [RENAME COLUMN](#rename-column)
5.  [ADD COLUMN](#add-column)
6.  [DROP COLUMN](#drop-column)
7.  [Change Column Type](#change-column-type)
8.  [Modify Column Constraints](#modify-column-constraints)
    *   [SET/DROP DEFAULT](#set-drop-default)
    *   [SET/DROP NOT NULL](#set-drop-not-null)
9.  [ADD/DROP PRIMARY KEY](#add-drop-primary-key)
10. [ADD/DROP UNIQUE Constraint](#add-drop-unique-constraint)
11. [Using IF EXISTS / IF NOT EXISTS](#using-if-exists--if-not-exists)
12. [ATTACH/DETACH Partitions](#attach-detach-partitions)
13. [Handling STRUCTs](#handling-structs)
14. [Limitations](#limitations)

---

## 1. Introduction

The `ALTER TABLE` statement in DuckDB allows you to modify the schema of an existing table. This is a powerful feature that enables you to evolve your database schema as your application requirements change. All modifications made with `ALTER TABLE` are transactional, meaning they will not be visible to other transactions until you commit, and they can be fully rolled back.

This guide will walk you through the various operations you can perform with `ALTER TABLE`, using a single, evolving example to demonstrate the effect of each command.

## 2. Creating the Initial Table

Let's start by creating a simple table called `employees` that we will modify throughout this guide.

```sql
CREATE TABLE employees (
    id INTEGER,
    first_name VARCHAR,
    last_name VARCHAR,
    start_date DATE
);
```

We can use the `DESCRIBE` command to view the current schema of our table.

```sql
DESCRIBE employees;
```

```
┌──────────────┬─────────────┬─────────┬─────────┬─────────┬─────────┐
│ column_name  │ column_type │  null   │   key   │ default │ extra   │
├──────────────┼─────────────┼─────────┼─────────┼─────────┼─────────┤
│ id           │ INTEGER     │ YES     │ NULL    │ NULL    │ NULL    │
│ first_name   │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ last_name    │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ start_date   │ DATE        │ YES     │ NULL    │ NULL    │ NULL    │
└──────────────┴─────────────┴─────────┴─────────┴─────────┴─────────┘
```

## 3. RENAME TABLE

You can rename a table using the `RENAME TO` clause.

```sql
ALTER TABLE employees RENAME TO staff;
```

Now, if you try to describe `employees`, you will get an error. The table is now named `staff`.

```sql
DESCRIBE staff;
```

For the rest of this guide, we will continue to use the `staff` table.

## 4. RENAME COLUMN

You can rename a column using the `RENAME COLUMN` or `RENAME` clause.

```sql
ALTER TABLE staff RENAME COLUMN start_date TO hire_date;
```

The `start_date` column is now named `hire_date`.

```sql
DESCRIBE staff;
```

```
┌──────────────┬─────────────┬─────────┬─────────┬─────────┬─────────┐
│ column_name  │ column_type │  null   │   key   │ default │ extra   │
├──────────────┼─────────────┼─────────┼─────────┼─────────┼─────────┤
│ id           │ INTEGER     │ YES     │ NULL    │ NULL    │ NULL    │
│ first_name   │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ last_name    │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ hire_date    │ DATE        │ YES     │ NULL    │ NULL    │ NULL    │
└──────────────┴─────────────┴─────────┴─────────┴─────────┴─────────┘
```

## 5. ADD COLUMN

You can add a new column to a table using the `ADD COLUMN` or `ADD` clause.

```sql
ALTER TABLE staff ADD COLUMN department VARCHAR;
```

A new `department` column has been added to the table. By default, it will be filled with `NULL` values.

```sql
DESCRIBE staff;
```

```
┌──────────────┬─────────────┬─────────┬─────────┬─────────┬─────────┐
│ column_name  │ column_type │  null   │   key   │ default │ extra   │
├──────────────┼─────────────┼─────────┼─────────┼─────────┼─────────┤
│ id           │ INTEGER     │ YES     │ NULL    │ NULL    │ NULL    │
│ first_name   │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ last_name    │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
│ hire_date    │ DATE        │ YES     │ NULL    │ NULL    │ NULL    │
│ department   │ VARCHAR     │ YES     │ NULL    │ NULL    │ NULL    │
└──────────────┴─────────────┴─────────┴─────────┴─────────┴─────────┘
```

You can also specify a `DEFAULT` value for the new column.

```sql
ALTER TABLE staff ADD COLUMN salary INTEGER DEFAULT 0;
```

## 6. DROP COLUMN

You can remove a column from a table using the `DROP COLUMN` or `DROP` clause.

```sql
ALTER TABLE staff DROP COLUMN department;
```

The `department` column has been removed.

## 7. Change Column Type

You can change the data type of a column using the `ALTER COLUMN ... TYPE` or `ALTER COLUMN ... SET DATA TYPE` clauses.

```sql
ALTER TABLE staff ALTER COLUMN id TYPE BIGINT;
```

The `id` column is now a `BIGINT`.

You can also provide a `USING` expression to specify how to convert the existing data to the new type. For example, to convert the `hire_date` column to a `VARCHAR` in the format 'YYYY-MM-DD':

```sql
ALTER TABLE staff ALTER hire_date TYPE VARCHAR USING(strftime(hire_date, '%Y-%m-%d'));
```

## 8. Modify Column Constraints

### SET/DROP DEFAULT

You can add or remove a `DEFAULT` value for a column.

```sql
ALTER TABLE staff ALTER COLUMN salary SET DEFAULT 50000;
```

Now, if you insert a new row without specifying a salary, it will default to `50000`.

To remove the default:
```sql
ALTER TABLE staff ALTER COLUMN salary DROP DEFAULT;
```

### SET/DROP NOT NULL

You can add or remove a `NOT NULL` constraint.

```sql
ALTER TABLE staff ALTER COLUMN last_name SET NOT NULL;
```

Now, you will not be able to insert a new row with a `NULL` value for `last_name`.

To remove the constraint:
```sql
ALTER TABLE staff ALTER COLUMN last_name DROP NOT NULL;
```

## 9. ADD/DROP PRIMARY KEY

You can add a `PRIMARY KEY` constraint to one or more columns.

```sql
ALTER TABLE staff ADD PRIMARY KEY (id);
```

The `id` column is now the primary key for the `staff` table.

To drop the primary key, you must know the name of the constraint. If you did not provide a name, DuckDB will generate one. You can find the name by inspecting the table schema.

```sql
-- This is not yet supported in DuckDB
-- ALTER TABLE staff DROP PRIMARY KEY;
```

## 10. ADD/DROP UNIQUE Constraint

You can add a `UNIQUE` constraint to one or more columns.

```sql
ALTER TABLE staff ADD UNIQUE (first_name, last_name);
```

This will ensure that no two employees have the same first and last name.

To drop a unique constraint, you must know the name of the constraint.

```sql
-- This is not yet supported in DuckDB
-- ALTER TABLE staff DROP CONSTRAINT constraint_name;
```

## 11. Using IF EXISTS / IF NOT EXISTS

Some `ALTER TABLE` operations support the `IF EXISTS` and `IF NOT EXISTS` clauses, which allow you to write more robust scripts that will not fail if the object you are trying to modify (or not modify) already exists (or doesn't).

For example, to add a column only if it does not already exist:
```sql
ALTER TABLE staff ADD COLUMN IF NOT EXISTS salary INTEGER;
```

To drop a column only if it exists:
```sql
ALTER TABLE staff DROP COLUMN IF EXISTS department;
```

## 12. ATTACH/DETACH Partitions

DuckDB allows you to manage partitioned tables using the `ATTACH` and `DETACH` clauses. This is a powerful feature for managing large datasets.

```sql
-- This feature is not yet fully documented with examples.
-- A more detailed guide will be created once the documentation is available.
```

## 13. Handling STRUCTs

`ALTER TABLE` can also be used to modify the schema of `STRUCT` columns. You can add, drop, or rename fields within a `STRUCT`.

```sql
CREATE TABLE events (
    event_id INTEGER,
    details STRUCT(name VARCHAR, location VARCHAR)
);

-- Add a new field
ALTER TABLE events ALTER COLUMN details ADD COLUMN event_date DATE;

-- Rename a field
ALTER TABLE events ALTER COLUMN details RENAME name TO event_name;

-- Drop a field
ALTER TABLE events ALTER COLUMN details DROP location;
```

## 14. Limitations

-   **Dependencies:** You cannot alter a column that has an index or is part of a multi-column `CHECK` constraint. You must drop the index or constraint first, alter the table, and then recreate it.
-   **Historical Data:** `ALTER COLUMN ... TYPE` may fail if the table has ever contained values that cannot be cast to the new type, even if those values have been deleted. The workaround is to create a new table with the desired schema and copy the data over.
-   **Dropping Constraints:** DuckDB does not yet support `DROP PRIMARY KEY` or `DROP CONSTRAINT`.

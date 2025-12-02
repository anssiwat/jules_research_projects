# DuckDB ALTER VIEW Statement Guide

## Table of Contents

1.  [Introduction](#introduction)
2.  [Syntax and Usage](#syntax-and-usage)
3.  [Practical Example](#practical-example)
    *   [Setup](#setup)
    *   [Renaming a View](#renaming-a-view)
    *   [Moving a View to a Different Schema](#moving-a-view-to-a-different-schema)
    *   [Using IF EXISTS](#using-if-exists)

---

## 1. Introduction

The `ALTER VIEW` statement in DuckDB allows you to change the properties of an existing view. Currently, the primary use of this statement is to rename a view or move it to a different schema. Like other schema-altering commands in DuckDB, `ALTER VIEW` is transactional, meaning the change will only be visible after the transaction is committed and can be rolled back.

## 2. Syntax and Usage

The syntax for renaming a view is straightforward:

```sql
ALTER VIEW [IF EXISTS] view_name RENAME TO new_view_name;
```

The syntax for moving a view to a different schema is:

```sql
ALTER VIEW [IF EXISTS] view_name SET SCHEMA new_schema_name;
```

The optional `IF EXISTS` clause will prevent an error from being thrown if the view you are trying to alter does not exist.

It's important to note that if you have other views that depend on the view you are altering, those dependent views will not be automatically updated. You will need to manually update them to refer to the new view name or schema.

## 3. Practical Example

### Setup

First, let's create a table and a view that we can work with.

```sql
CREATE TABLE employees (
    id INTEGER,
    first_name VARCHAR,
    last_name VARCHAR,
    department VARCHAR,
    salary INTEGER
);

INSERT INTO employees VALUES
    (1, 'John', 'Doe', 'Engineering', 80000),
    (2, 'Jane', 'Smith', 'Marketing', 90000),
    (3, 'Peter', 'Jones', 'Engineering', 85000);

CREATE VIEW engineering_staff AS
SELECT first_name, last_name, salary
FROM employees
WHERE department = 'Engineering';
```

You can query the view to see the results:

```sql
SELECT * FROM engineering_staff;
```

```
┌────────────┬───────────┬────────┐
│ first_name │ last_name │ salary │
│  varchar   │  varchar  │ int32  │
├────────────┼───────────┼────────┤
│ John       │ Doe       │  80000 │
│ Peter      │ Jones     │  85000 │
└────────────┴───────────┴────────┘
```

### Renaming a View

Now, let's rename the `engineering_staff` view to `engineers`.

```sql
ALTER VIEW engineering_staff RENAME TO engineers;
```

The view has now been renamed. If you try to query the old view name, you will get an error. You can query the new view name to confirm the change:

```sql
SELECT * FROM engineers;
```

```
┌────────────┬───────────┬────────┐
│ first_name │ last_name │ salary │
│  varchar   │  varchar  │ int32  │
├────────────┼───────────┼────────┤
│ John       │ Doe       │  80000 │
│ Peter      │ Jones     │  85000 │
└────────────┴───────────┴────────┘
```

### Moving a View to a Different Schema

You can also use `ALTER VIEW` to move a view to a different schema. First, let's create a new schema:

```sql
CREATE SCHEMA production;
```

Now, let's move the `engineers` view to the `production` schema:

```sql
ALTER VIEW engineers SET SCHEMA production;
```

The view is now in the `production` schema. You can query it using its fully qualified name:

```sql
SELECT * FROM production.engineers;
```

### Using IF EXISTS

If you try to alter a view that does not exist, you will get an error.

```sql
ALTER VIEW non_existent_view RENAME TO new_name;
```

To avoid this, you can use the `IF EXISTS` clause. If the view does not exist, the command will do nothing instead of throwing an error.

```sql
ALTER VIEW IF EXISTS non_existent_view RENAME TO new_name;
```

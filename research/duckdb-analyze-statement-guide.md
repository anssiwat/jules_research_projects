# DuckDB ANALYZE Statement Guide

## Table of Contents

1.  [Introduction](#introduction)
    *   [What is ANALYZE?](#what-is-analyze)
2.  [Syntax and Usage](#syntax-and-usage)
3.  [Practical Example](#practical-example)
    *   [Setup](#setup)
    *   [Query Plan Before ANALYZE](#query-plan-before-analyze)
    *   [Running ANALYZE](#running-analyze)
    *   [Query Plan After ANALYZE](#query-plan-after-analyze)
4.  [When to Use ANALYZE](#when-to-use-analyze)

---

## 1. Introduction

### What is ANALYZE?

The `ANALYZE` statement in DuckDB is a crucial tool for query optimization. Its primary function is to compute and update the statistics for tables in the database. These statistics provide the query optimizer with detailed information about the data distribution within the tables, such as the number of distinct values and the presence of `NULL`s.

While DuckDB automatically collects some statistics, `ANALYZE` performs a more thorough collection. The query optimizer uses these detailed statistics to make more informed decisions, particularly for choosing the most efficient join order and join algorithms in complex queries. By providing accurate statistics, `ANALYZE` can significantly improve the performance of your queries, especially those involving multiple joins or large tables.

## 2. Syntax and Usage

The syntax for the `ANALYZE` statement is straightforward. You can either analyze all tables in the current schema or specify a particular table.

To analyze all tables in the database:
```sql
ANALYZE;
```

To analyze a specific table:
```sql
ANALYZE table_name;
```

Running `ANALYZE` recalculates the statistics and stores them in the system catalog. The optimizer will then use this updated information the next time a query is planned.

## 3. Practical Example

To demonstrate the impact of `ANALYZE`, let's walk through a scenario where it helps the query optimizer choose a better join plan.

### Setup

First, let's create two tables: `users`, which will be a large table, and `premium_users`, a much smaller table containing a subset of user IDs.

```sql
CREATE TABLE users (
    user_id INTEGER,
    user_name VARCHAR
);

CREATE TABLE premium_users (
    user_id INTEGER
);

-- Insert 1 million users into the 'users' table
INSERT INTO users SELECT i, 'user_' || i FROM range(1000000) t(i);

-- Insert 100 premium users
INSERT INTO premium_users SELECT i FROM range(100) t(i);
```

Now, let's consider a simple query that joins these two tables to find the names of the premium users.

```sql
SELECT u.user_name
FROM users u
JOIN premium_users p ON u.user_id = p.user_id;
```

### Query Plan Before ANALYZE

Without up-to-date statistics, the query optimizer might make a suboptimal decision. We can inspect the query plan using the `EXPLAIN` statement.

```sql
EXPLAIN SELECT u.user_name
FROM users u
JOIN premium_users p ON u.user_id = p.user_id;
```

The output might look something like this (the exact plan can vary):

```
┌───────────────────────────────────────────────────┐
│                     QUERY PLAN                    │
├───────────────────────────────────────────────────┤
│                  PROJECTION [1]                   │
│          ┌──────────┴──────────┐                  │
│        [INFOSEPARATOR]         [INFOSEPARATOR]    │
│            user_name           u.user_id          │
│   ┌────────────┴───────────┐                      │
│        HASH_JOIN [1]         ...                  │
│   ...                                             │
│     Build: users (1000000)                        │
│     Probe: premium_users (100)                    │
│   ...                                             │
└───────────────────────────────────────────────────┘
```
In this suboptimal plan, the optimizer has chosen to build the hash table on the `users` table, which is very large, and then probe it with the smaller `premium_users` table. Building a hash table on one million rows is much less efficient than building it on one hundred rows.

### Running ANALYZE

Now, let's run `ANALYZE` to update the table statistics.

```sql
ANALYZE;
```
This command will scan the tables and collect detailed statistics about their contents.

### Query Plan After ANALYZE

With accurate statistics, the optimizer now knows that `premium_users` is significantly smaller than `users`. Let's look at the query plan again.

```sql
EXPLAIN SELECT u.user_name
FROM users u
JOIN premium_users p ON u.user_id = p.user_id;
```

The new, optimized query plan will look like this:

```
┌───────────────────────────────────────────────────┐
│                     QUERY PLAN                    │
├───────────────────────────────────────────────────┤
│                  PROJECTION [1]                   │
│          ┌──────────┴──────────┐                  │
│        [INFOSEPARATOR]         [INFOSEPARATOR]    │
│            user_name           u.user_id          │
│   ┌────────────┴───────────┐                      │
│        HASH_JOIN [1]         ...                  │
│   ...                                             │
│     Build: premium_users (100)                    │
│     Probe: users (1000000)                        │
│   ...                                             │
└───────────────────────────────────────────────────┘
```
As you can see, the optimizer has now correctly chosen to build the hash table on the much smaller `premium_users` table and probe it with the `users` table. This is a far more efficient join strategy that will result in a faster query execution time.

## 4. When to Use ANALYZE

While DuckDB's query optimizer is highly effective out of the box, running `ANALYZE` is recommended in the following situations:

-   **After Large Data Loads:** If you have just inserted a large volume of data into your tables.
-   **After Significant Data Changes:** If you have performed bulk `UPDATE` or `DELETE` operations that have substantially changed the data distribution in your tables.
-   **Before Complex Queries:** If you are about to run complex analytical queries with multiple joins, ensuring that the statistics are up-to-date can prevent poor query plan choices.

In general, if you notice that a join query is performing slower than expected, running `ANALYZE` is a good first step in troubleshooting and optimization.

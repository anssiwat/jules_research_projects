# A Comprehensive Guide to Working with Nested Data Types in DuckDB

DuckDB is a high-performance analytical database that is known for its speed and efficiency. One of the features that makes DuckDB so powerful is its support for nested data types, which allow you to store and query complex, semi-structured data with ease. In this guide, we will explore how to work with Maps, Lists, JSON, and Structs in DuckDB, and provide you with the knowledge and tools you need to effectively manage and analyze nested data.

## Table of Contents

- [Introduction to Nested Data Types](#introduction-to-nested-data-types)
- [Maps](#maps)
- [Lists](#lists)
- [JSON](#json)
- [Structs](#structs)
- [Performance Considerations](#performance-considerations)
- [Conclusion](#conclusion)

## Introduction to Nested Data Types

Nested data types, also known as complex or composite data types, are data types that can contain other data types. This allows you to represent hierarchical or multi-level data structures in a single column, which can be particularly useful when working with data from sources like JSON or Parquet.

DuckDB supports a rich set of nested data types, including:

- **Maps**: An unordered collection of key-value pairs.
- **Lists**: An ordered collection of values of the same type.
- **JSON**: A semi-structured data format that can contain a mix of objects, arrays, and scalar values.
- **Structs**: An ordered collection of named fields, each with its own data type.

In the following sections, we will explore each of these data types in detail, and provide you with practical examples of how to create, query, and manipulate them in DuckDB.

## Maps

Maps are a powerful data type that allow you to store and query key-value pairs. They are similar to `STRUCT`s, but with a key difference: maps do not need to have the same keys present for each row. This makes them ideal for semi-structured data where the schema can vary.

A map must have a single type for all its keys and a single type for all its values, though the key and value types do not need to be the same. Keys cannot be duplicated within a map.

### Creating Maps

DuckDB offers several ways to create maps.

**1. Bracket Syntax**

The most common way is to use the `MAP` keyword with bracket syntax:
```sql
-- A map with VARCHAR keys and INTEGER values
SELECT MAP {'key1': 10, 'key2': 20, 'key3': 30};

-- A map with INTEGER keys and NUMERIC values
SELECT MAP {1: 42.001, 5: -32.1};
```

**2. From Lists**

You can also create a map from two lists (one for keys, one for values):
```sql
SELECT MAP(['key1', 'key2', 'key3'], [10, 20, 30]);
```

**3. From Entries**

The `map_from_entries` function creates a map from a list of structs (or entries):
```sql
SELECT map_from_entries([('key1', 10), ('key2', 20), ('key3', 30)]);
```

**4. Maps with Nested Types**

The keys and values in a map can themselves be nested types:
```sql
-- A map with a LIST of VARCHARs as keys and a LIST of DOUBLEs as values
SELECT MAP {['a', 'b']: [1.1, 2.2], ['c', 'd']: [3.3, 4.4]};
```

**5. Maps in Tables**

You can define a column as a `MAP` type in a table definition:
```sql
CREATE TABLE user_metadata (
    user_id BIGINT,
    properties MAP(VARCHAR, VARCHAR)
);

INSERT INTO user_metadata VALUES
    (1, MAP {'country': 'USA', 'source': 'organic'}),
    (2, MAP {'country': 'Canada', 'source': 'referral', 'premium_user': 'true'});
```

### Querying Maps

You can retrieve values from maps using bracket notation or the `map_extract` function.

**Bracket Notation**

Bracket notation is a concise way to extract a value. If the key does not exist, it returns `NULL`.
```sql
SELECT
    properties['country'] AS country,
    properties['premium_user'] AS premium_status
FROM user_metadata;
/*
┌─────────┬────────────────┐
│ country │ premium_status │
│ varchar │    varchar     │
├─────────┼────────────────┤
│ USA     │ NULL           │
│ Canada  │ true           │
└─────────┴────────────────┘
*/
```

**`map_extract` Function**

The `map_extract` function (and its synonym `element_at`) can also be used. It returns a list containing the value if the key is found, and an empty list `[]` if the key is not found.
```sql
SELECT
    map_extract(properties, 'country') AS country,
    map_extract(properties, 'premium_user') AS premium_status
FROM user_metadata;
/*
┌─────────┬────────────────┐
│ country │ premium_status │
│ varchar │    varchar     │
├─────────┼────────────────┤
│ [USA]   │ []             │
│ [Canada]│ [true]         │
└─────────┴────────────────┘
*/
```

### Map Functions

DuckDB provides a rich set of functions for working with maps, including:

- `map_keys(map)`: Returns a list of all keys in the map.
- `map_values(map)`: Returns a list of all values in the map.
- `cardinality(map)`: Returns the number of key-value pairs in the map.

For a full list of functions, refer to the [official DuckDB documentation on Map Functions](https://duckdb.org/docs/stable/sql/functions/map).

## Lists

A `LIST` is an ordered collection of values of the same type. Lists are flexible in that each row can have a list of a different length. This makes them suitable for storing arrays of values, such as sensor readings, tags, or historical data.

### Creating Lists

Lists can be created using bracket notation or the `list_value` function.

**1. Bracket Notation**

The most straightforward way to create a list is with brackets `[]`:
```sql
-- A list of integers
SELECT [1, 2, 3];

-- A list of strings, including a NULL
SELECT ['duck', 'goose', NULL, 'heron'];

-- A list of lists
SELECT [['duck', 'goose'], NULL, ['frog', 'toad'], []];
```

**2. `list_value` function**

The `list_value` function provides an alternative syntax:
```sql
SELECT list_value(1, 2, 3);
```

**3. Lists in Tables**

To store lists in a table, you can define a column with a `LIST` type, denoted by `[]` after the data type.
```sql
CREATE TABLE product_tags (
    product_id BIGINT,
    tags VARCHAR[]
);

INSERT INTO product_tags VALUES
    (101, ['electronics', 'audio', 'headphones']),
    (102, ['electronics', 'camera']),
    (103, ['books', 'fiction']);
```

### Querying Lists

You can access list elements by index, slice lists, and use a variety of functions to work with them.

**Indexing**

DuckDB uses 1-based indexing for lists. You can use bracket notation to access elements. Negative indices count from the end of the list.
```sql
SELECT tags[1] AS first_tag, tags[-1] AS last_tag FROM product_tags;
/*
┌─────────────┬────────────┐
│  first_tag  │  last_tag  │
│   varchar   │  varchar   │
├─────────────┼────────────┤
│ electronics │ headphones │
│ electronics │ camera     │
│ books       │ fiction    │
└─────────────┴────────────┘
*/
```

**Slicing**

You can extract a sublist (a "slice") using `[start:end]` notation. The slice is inclusive of the start and end indices.
```sql
SELECT tags[1:2] AS first_two_tags FROM product_tags;
/*
┌───────────────────────────┐
│      first_two_tags       │
│         varchar[]         │
├───────────────────────────┤
│ [electronics, audio]      │
│ [electronics, camera]     │
│ [books, fiction]          │
└───────────────────────────┘
*/
```

### Unnesting Lists

A common operation is to "unnest" a list, which expands each element of the list into its own row. This is useful for aggregations or joins.
```sql
SELECT product_id, unnest(tags) AS tag
FROM product_tags;
/*
┌────────────┬─────────────┐
│ product_id │     tag     │
│   int64    │   varchar   │
├────────────┼─────────────┤
│        101 │ electronics │
│        101 │ audio       │
│        101 │ headphones  │
│        102 │ electronics │
│        102 │ camera      │
│        103 │ books       │
│        103 │ fiction     │
└────────────┴─────────────┘
*/
```
This allows you to perform aggregations like counting the occurrences of each tag:
```sql
SELECT unnest(tags) AS tag, COUNT(*) AS count
FROM product_tags
GROUP BY tag
ORDER BY count DESC;
```

### List Functions

DuckDB provides many functions for list manipulation, including:

- `list_contains(list, element)`: Checks if a list contains an element.
- `len(list)` or `cardinality(list)`: Returns the number of elements in a list.
- `list_append(list, element)`: Appends an element to the end of a list.
- `list_sort(list)`: Sorts the elements of a list.

For a complete list, see the [official DuckDB documentation on List Functions](https://duckdb.org/docs/stable/sql/functions/list).

## JSON

JSON (JavaScript Object Notation) is a ubiquitous format for semi-structured data. DuckDB's `json` extension, which is auto-loaded on first use, provides a powerful set of tools for ingesting, storing, and querying JSON data directly in SQL.

### Loading JSON Data

DuckDB can read JSON data directly from files, making it easy to analyze large datasets without a complex ETL pipeline.

**From a Local File**

The `read_json` function can automatically infer the schema of a JSON file.
```sql
-- Assumes a file named 'ducks.json' exists with a JSON array of objects
SELECT * FROM 'ducks.json';

-- You can also specify the schema for more control
SELECT *
FROM read_json('ducks.json',
               format = 'array',
               columns = {id: 'VARCHAR',
                          color: 'VARCHAR',
                          name: 'STRUCT(firstName VARCHAR, lastName VARCHAR)'});
```

**From an API**

DuckDB can even query JSON data directly from an HTTP endpoint.
```sql
-- Query the TVmaze API for shows about ducks
SELECT show.name, show.type, show.summary
FROM read_json('https://api.tvmaze.com/search/shows?q=duck', auto_detect=true);
```

### The `JSON` Data Type

For storing JSON within a table, DuckDB provides a `JSON` data type. This is essentially a `VARCHAR` with special validation to ensure it contains valid JSON.
```sql
CREATE TABLE events (
    event_id BIGINT,
    ts TIMESTAMP,
    payload JSON
);

INSERT INTO events VALUES
    (1, now(), '{ "user_id": 123, "action": "login", "details": {"ip": "192.168.1.1"} }'),
    (2, now(), '{ "user_id": 456, "action": "purchase", "details": {"product_id": "abc", "amount": 99.99} }');
```

### Querying JSON

DuckDB uses `JSONPath` expressions to navigate the structure of a JSON object. You can extract data using the `->` and `->>` operators or the `json_extract` function.

**`->` and `->>` Operators**

- `->`: Extracts a value as a JSON object.
- `->>`: Extracts a value as a text/`VARCHAR`.

```sql
-- Extract the user_id as a VARCHAR
SELECT payload->>'$.user_id' AS user_id
FROM events;

-- Extract the nested 'details' object as JSON
SELECT payload->'$.details' AS details
FROM events;
```

**0-based Indexing**

A key point to remember is that DuckDB uses **0-based indexing** for JSON arrays, in contrast to the 1-based indexing for `LIST`s and `ARRAY`s.
```sql
SELECT json_extract('[10, 20, 30]', '$[0]'); -- returns 10
```

### Complex JSON Queries

**Unnesting**

For complex, nested JSON, the `unnest` function is invaluable. It can transform a nested array of objects into a relational format.
```json
-- ducks-example.json
{
  "ducks": [
    { "name": "Quackmire", "color": "green", "actions": ["swimming", "waddling"] },
    { "name": "Feather Locklear", "color": "yellow", "actions": ["sunbathing"] }
  ],
  "totalDucks": 2
}
```
```sql
SELECT unnest(ducks, recursive:=true) AS ducks
FROM 'ducks-example.json';
/*
┌──────────────────┬─────────┬──────────────────────────┐
│       name       │  color  │         actions          │
│     varchar      │ varchar │        varchar[]         │
├──────────────────┼─────────┼──────────────────────────┤
│ Quackmire        │ green   │ [swimming, waddling]     │
│ Feather Locklear │ yellow  │ [sunbathing]             │
└──────────────────┴─────────┴──────────────────────────┘
*/
```
This allows you to easily query the nested data using standard SQL.

### JSON Functions

DuckDB includes a wide array of JSON functions for creation, manipulation, and extraction, such as:

- `json_valid(json)`: Checks if a string is valid JSON.
- `json_array_length(json)`: Returns the length of a JSON array.
- `json_keys(json)`: Returns the top-level keys of a JSON object.
- `to_json(value)` and `from_json(json, type)`: Convert to and from JSON.

For a comprehensive overview, see the [official DuckDB documentation on JSON Functions](https://duckdb.org/docs/stable/data/json/json_functions).

## Structs

A `STRUCT` is an ordered collection of named fields, much like a row in a table. Each field, or "entry," has a name (a key) and a value, and each value can have a different data type. Unlike `MAP`s, all `STRUCT`s in a given column must have the same set of keys. This fixed schema allows for highly efficient, vectorized execution.

### Creating Structs

DuckDB provides a few ways to create `STRUCT`s.

**1. Curly Brace Notation**

The most common method is the curly brace `{}` notation:
```sql
-- A simple struct
SELECT {'x': 1, 'y': 2, 'z': 3};

-- A struct with mixed types
SELECT {'key1': 'string', 'key2': 1, 'key3': 12.345};

-- A struct containing other nested types
SELECT {
    'birds': {'yes': 'duck', 'maybe': 'goose'},
    'amphibians': ['frog', 'salamander']
};
```

**2. `struct_pack` Function**

The `struct_pack` function offers an alternative, more explicit syntax:
```sql
SELECT struct_pack(key1 := 'value1', key2 := 42);
```

**3. `row` Function**

The `row` function can convert multiple columns into a single `STRUCT` column.
```sql
CREATE TABLE location_data (s STRUCT(city VARCHAR, country VARCHAR));
INSERT INTO location_data VALUES (row('Amsterdam', 'Netherlands'));
```

### Querying Structs

Values within a `STRUCT` can be accessed using dot notation or the `struct_extract` function.

**Dot Notation**

This is the most convenient way to access a field.
```sql
CREATE TABLE users AS SELECT 1 as id, {'first': 'Jules', 'last': 'Cool'} as name;

SELECT id, name.first FROM users;
/*
┌───┬───────┐
│id │ first │
├───┼───────┤
│ 1 │ Jules │
└───┴───────┘
*/
```
If a key contains a space or special character, you can wrap it in double quotes: `my_struct."my key"`.

**`struct_extract` Function**

This function achieves the same result:
```sql
SELECT id, struct_extract(name, 'first') FROM users;
```

### Unnesting Structs

The `unnest` function is a powerful feature that can expand the fields of a `STRUCT` into their own columns. The `*` notation provides a convenient shortcut.

```sql
SELECT name.* FROM users;
/*
┌───────┬───────┐
│ first │ last  │
├───────┼───────┤
│ Jules │ Cool  │
└───────┴───────┘
*/

-- You can also exclude or replace fields
SELECT name.* EXCLUDE ('last') FROM users;
```

### Updating the Schema

DuckDB allows you to modify the schema of a `STRUCT` column using `ALTER TABLE` commands.

```sql
ALTER TABLE users ADD COLUMN name.middle VARCHAR;
ALTER TABLE users DROP COLUMN name.last;
ALTER TABLE users RENAME name.first TO first_name;
```

### Struct Functions

DuckDB offers several functions for working with `STRUCT`s, including:

- `struct_insert(struct, key, value)`: Adds a new field to a struct.
- `struct_update(struct, key, value)`: Updates an existing field or adds a new one.

For more details, see the [official DuckDB documentation on Struct Functions](https://duckdb.org/docs/stable/sql/functions/struct).

## Performance Considerations

Working with nested data types in DuckDB is highly efficient, but there are still best practices to follow to ensure optimal performance.

- **`STRUCT` vs. `MAP`**: When your data has a consistent, known schema, always prefer `STRUCT` over `MAP`. The fixed structure of a `STRUCT` allows DuckDB's vectorized engine to operate at maximum efficiency. Use `MAP` for truly semi-structured data where keys can vary between rows.
- **`unnest` for Filtering**: When you need to filter based on values within a list or a nested object, it is often more performant to `unnest` the data first. This allows the query optimizer to better utilize indexes and apply filters efficiently.
- **`JSON` vs. Native Types**: The `JSON` type is a `VARCHAR` and requires parsing at query time. If you consistently extract the same fields from a JSON object, consider transforming the data into native `STRUCT`, `MAP`, and `LIST` columns. This pre-parsing will significantly speed up query execution.
- **Avoid Deep Nesting When Possible**: While DuckDB can handle deeply nested data, overly complex structures can make queries harder to write and potentially slower to execute. If possible, flatten your data to a more relational format when it makes sense for your query patterns.

## Conclusion

DuckDB's support for nested data types like `MAP`, `LIST`, `JSON`, and `STRUCT` provides a powerful bridge between the relational world of SQL and the semi-structured nature of modern data formats. By understanding the strengths of each type and leveraging DuckDB's rich set of functions, you can build efficient and elegant data analysis pipelines.

This guide has provided a comprehensive overview of how to create, query, and manage these complex data types. With these tools in hand, you are now well-equipped to tackle a wide range of data challenges, from analyzing API responses and log files to building complex analytical models.

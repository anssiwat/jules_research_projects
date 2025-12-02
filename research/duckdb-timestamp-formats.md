# DuckDB Timestamp Formats

DuckDB supports several timestamp formats with varying precision. The standard format is based on ISO 8601.

## Timestamp Types

| Name | Aliases | Description |
|---|---|---|
| `TIMESTAMP_NS` | | Naive timestamp with nanosecond precision |
| `TIMESTAMP` | `DATETIME`, `TIMESTAMP WITHOUT TIME ZONE` | Naive timestamp with microsecond precision |
| `TIMESTAMP_MS` | | Naive timestamp with millisecond precision |
| `TIMESTAMP_S` | | Naive timestamp with second precision |
| `TIMESTAMPTZ` | `TIMESTAMP WITH TIME ZONE` | Time zone aware timestamp with microsecond precision |

## ISO 8601 Format

The standard string format for timestamps is `YYYY-MM-DD hh:mm:ss[.zzzzzzzzz][+-TT[:tt]]`.

## Special Values

- `epoch`: Represents `1970-01-01 00:00:00[+00]`
- `infinity`: A timestamp later than all others.
- `-infinity`: A timestamp earlier than all others.

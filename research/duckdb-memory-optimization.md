# DuckDB Memory Optimization

This document summarizes best practices for optimizing DuckDB memory usage, based on the official documentation and community guides.

## Performance Tuning

- **`preserve_insertion_order`**: For large datasets, disabling this option can reduce memory usage by allowing the system to reorder results.
- **Parallelism**: DuckDB parallelizes workloads based on row groups. For queries to run on `k` threads, at least `k * 122,880` rows are required.
- **Thread Management**: Manually limit the number of threads if DuckDB launches too many, which can cause slowdowns.
- **Out-of-Core Processing**: DuckDB supports larger-than-memory workloads by spilling to disk. A temporary directory is created for this purpose, which can be configured.
- **Blocking Operators**: `GROUP BY`, `JOIN`, `ORDER BY`, and windowing functions are memory-intensive. While DuckDB supports out-of-core processing for these, complex queries with multiple blocking operators may still cause out-of-memory exceptions.

## Profiling and Optimization

- **`EXPLAIN` and `EXPLAIN ANALYZE`**: Use these commands to inspect query plans and identify performance bottlenecks.
- **Prepared Statements**: Use for repeatedly running small queries with different parameters to improve performance.
- **Remote Files**:
  - Increase the number of threads (2-5 times the number of CPU cores) to improve parallelism when querying remote files.
  - Avoid `SELECT *` to reduce the amount of data read.
  - Apply filters on remote Parquet files to minimize data scanning.
  - Use caching for remote data to improve performance.

## Connection Management

- **Reuse Connections**: Reusing the same database connection can improve performance by avoiding overhead and preserving in-memory caches.

## Storage

- **Persistent vs. In-Memory Tables**: Compression is applied by default on persistent databases, which can make them faster than uncompressed in-memory databases.
- **On-Disk Databases**: For memory-intensive workloads, use on-disk databases to allow DuckDB to spill to disk.
- **Fast Storage**: When spilling to disk, use fast storage and avoid network storage for better performance.

## Configuration

- **Memory Limit**: The memory limit can be set via environment variables or configuration files. The minimum required memory is 128MB per thread, with a recommendation of around 5GB per thread for optimal performance.
- **Thread Limit**: The number of threads can be configured to match the available CPU cores.

## Salting

- **Salting**: In some cases, salting blocking rules can help achieve full CPU utilization, especially on machines with a high core count or when working with a small number of input rows. However, reducing salting can also help reduce memory usage.

# Performance Review

**Scope note:** This review applies to code operating at real scale — cloud services, large dataset processing, high-throughput pipelines, database queries on large tables. It does NOT apply to typical application code iterating small in-memory collections. If the code under review is standard CRUD on small datasets, most of these checks are irrelevant. Focus on what matters for the actual workload.

## What to Look For

### Algorithm Complexity at Data Scale
- O(n^2) or worse algorithms on datasets that grow with users/data (not fixed-size config arrays)
- Nested loops where both dimensions scale independently
- Repeated linear searches through large collections (should be a hash map/set lookup)
- Sorting when only the top-N elements are needed (use a heap or partial sort)
- String concatenation in loops building large strings (use a builder/buffer/join)

### Database Query Patterns
- **N+1 queries**: Loading a list, then querying for related data one row at a time. Use eager loading, joins, or batch queries.
- **Missing indexes for query patterns at scale**: Queries that scan large tables without hitting an index. Check EXPLAIN plans for new query patterns.
- **Unbounded result sets**: Queries that return all matching rows with no LIMIT. A collection that starts at 100 rows and grows to 1M rows will eventually cause memory pressure and timeouts.
- **Full table scans in hot paths**: Queries that run on every request and scan the full table.
- **Over-fetching columns**: `SELECT *` when only a few columns are needed, especially with large text/blob columns.
- **Write amplification**: Updating an entire row when only one column changed. Matters at scale with wide rows.

### Memory Usage
- Loading large datasets entirely into memory (reading a whole file, fetching all DB rows, loading a full API response) when streaming or pagination would work
- Unbounded caches that grow with data volume and never evict
- Large object allocation in tight loops (allocate outside the loop, reuse)
- Holding references to large objects longer than necessary (prevents garbage collection)
- Buffer sizing: fixed large buffers when the typical payload is small, or undersized buffers that force constant reallocation

### Concurrency
- **Connection pool sizing**: Database/HTTP connection pools that are too small for the workload (blocking under load) or too large (resource exhaustion)
- **Lock contention**: Shared locks held during I/O operations, locks with wide scope that serialize concurrent requests
- **Async/await correctness**: Blocking calls inside async functions that hold up the event loop. Sequential awaits that could be concurrent (Promise.all, asyncio.gather).
- **Thread safety**: Shared mutable state accessed from multiple threads without synchronization
- **Deadlock potential**: Multiple locks acquired in inconsistent order across code paths

### Network Patterns
- **Chatty service calls**: Multiple sequential calls to the same service when one batched call would work
- **Missing batching**: Processing items one at a time over the network when the API supports batch operations
- **No retry/backoff**: Network calls that fail once and give up, or retry immediately in a tight loop (no exponential backoff)
- **Missing timeouts**: Network calls with no timeout — a slow dependency hangs the caller indefinitely
- **Large payloads**: Transferring large objects between services when only a subset of data is needed

### Caching Strategy
- **Appropriate cache levels**: Caching at the right layer (application memory, distributed cache, CDN) for the access pattern
- **Invalidation correctness**: Cache entries are invalidated or expired when underlying data changes. Stale cache bugs are hard to diagnose.
- **Cache stampede risk**: When a popular cache entry expires, many concurrent requests all try to rebuild it simultaneously. Use locks, early refresh, or staggered TTLs.
- **Cache key design**: Keys should include all parameters that affect the cached value. Missing a parameter = serving wrong data to some users.

### Library and Service-Specific Performance
- Apply your knowledge of performance characteristics for the specific libraries, databases, and cloud services being used
- Framework-specific pitfalls: ORM lazy loading defaults, serialization overhead, middleware ordering
- Cloud service limits: API rate limits, throughput provisioning, connection limits
- Database engine specifics: PostgreSQL VACUUM needs, MySQL query cache behavior, Redis memory management

## Common Issues

- **Pagination afterthought**: Code works with test data (20 rows) but queries all records. When production hits 100K rows, the endpoint times out and uses 2GB of memory.
- **N+1 in list views**: Fetching a list of 50 items, then running 50 individual queries for related data. A single JOIN or IN query replaces 51 queries with 1-2.
- **Sync I/O in async context**: Calling a synchronous HTTP client or file read inside an async handler. Blocks the entire event loop, making the server handle one request at a time.
- **Unbounded cache growth**: `cache[key] = value` with no eviction policy. Memory usage grows linearly with data volume until the process is killed.
- **Sequential API calls that could be parallel**: Calling service A, waiting, then calling service B, waiting. If the calls are independent, run them concurrently.
- **Logging in hot loops**: Logging at INFO/DEBUG level inside a loop that runs millions of times. The logging overhead dominates the actual work.
- **Missing connection pooling**: Creating a new database or HTTP connection per request instead of using a pool. Connection setup overhead adds up under load.
- **Premature optimization on cold paths**: Optimizing a migration script or admin-only endpoint that runs once a day. Focus on hot paths.

## Severity Guidance

**Critical:**
- N+1 query patterns on unbounded collections in hot paths (will cause outages at scale)
- Unbounded result sets in production queries (memory/timeout bomb)
- Blocking I/O in async context (server becomes single-threaded under load)
- Missing timeouts on network calls to external services (hung dependency = hung service)
- Deadlock potential in production code paths

**Moderate:**
- O(n^2) algorithms on datasets that will grow but aren't huge yet
- Unbounded caches with no eviction
- Sequential network calls that could be parallelized (latency impact)
- Missing connection pooling (performance degradation under load)
- Over-fetching from database (SELECT * on wide tables in hot paths)

**Mild:**
- Minor allocation inefficiencies in non-hot paths
- Cache key design that works but could be more efficient
- Logging verbosity in moderate-frequency code paths
- Missing batch operations where individual calls work acceptably at current scale

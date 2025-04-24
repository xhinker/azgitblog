# How to monitor PostgreSQL and identify the hot spot

- [How to monitor PostgreSQL and identify the hot spot](#how-to-monitor-postgresql-and-identify-the-hot-spot)
    - [1. See active queries and source IP addresses](#1-see-active-queries-and-source-ip-addresses)
        - [1.1 Take a look at the activities](#11-take-a-look-at-the-activities)
        - [1.2 Manually kill idle connections longer than 1 hour](#12-manually-kill-idle-connections-longer-than-1-hour)
        - [1.3 Kill idle connections automatically](#13-kill-idle-connections-automatically)
    - [2. Use `pg_stat_statements` to see query resource usage](#2-use-pg_stat_statements-to-see-query-resource-usage)
        - [2.1 Enable the extension:](#21-enable-the-extension)
        - [2.2 Identify CPU-Intensive Queries:](#22-identify-cpu-intensive-queries)

Today, the Ubuntu VM that host PostgreSQL CPU usage surge to 100%. Need to find out what which query or component caused the CPU usage surge.

## 1. See active queries and source IP addresses

### 1.1 Take a look at the activities
View active queries and their process details:

```sql
SELECT 
    pid
    , usename
    , application_name
    , client_addr
    , query
    , state
    , backend_type 
FROM 
    pg_stat_activity 
WHERE 1 = 1
    and state = 'active'  -- Exclude idle connections
ORDER BY 
    query_start DESC;  -- Most recent first
```

See idle activities:

```sql 
SELECT 
    pid
    , usename
    , application_name
    , client_addr
    , query
    , state
    , backend_type 
FROM 
    pg_stat_activity 
WHERE 1 = 1
    and state ('idle', 'idle in transaction')
ORDER BY 
    query_start DESC;  -- Most recent first
```

### 1.2 Manually kill idle connections longer than 1 hour

```sql
SELECT 
    pg_terminate_backend(pid) 
FROM 
    pg_stat_activity 
WHERE 
    state = 'idle' 
    AND state_change < NOW() - INTERVAL '1 hour';
```

### 1.3 Kill idle connections automatically

Add to postgresql.conf:
```ini
idle_in_transaction_session_timeout = 60000  # Kill idle-in-transaction after 60s (in ms)
idle_session_timeout = 30000                 # Kill idle connections after 30s (optional)
```

Restart PostgreSQL: 
```sh
sudo systemctl restart postgresql
```

## 2. Use `pg_stat_statements` to see query resource usage

Enable the pg_stat_statements extension to track query performance metrics:

### 2.1 Enable the extension:

* Add to postgresql.conf:
```ini
shared_preload_libraries = 'pg_stat_statements'
pg_stat_statements.track = all
```

The `pg_stat_statements.track` settin  g in PostgreSQL controls which kinds of SQL statements are tracked by the pg_stat_statements extension, which is used for query performance monitoring.

Possible Values:
`none` – No statements are tracked.

`top (default)` – Only top-level SQL statements (i.e., not inside PL/pgSQL functions or other procedural code) are tracked.

`all` – Both top-level and nested statements are tracked, including those executed within functions, triggers, or other procedural code.


* Restart PostgreSQL: `sudo systemctl restart postgresql`
* Create the extension in your database:
```sql
CREATE EXTENSION pg_stat_statements;
```

### 2.2 Identify CPU-Intensive Queries:

* See which query used the most time, and call number.
```sql
SELECT 
    queryid
    , query
    , ROUND(total_exec_time) as total_exec_time
    , ROUND(mean_exec_time) as mean_exec_time
    , calls 
FROM 
    g_stat_statements 
ORDER BY 
    total_exec_time DESC 
LIMIT 10;
```

* See which query has the highest average call time

```sql
SELECT 
    query
    , calls
    , total_exec_time
    , total_exec_time/calls AS avg_ms
FROM   
    pg_stat_statements
ORDER BY 
    total_exec_time DESC
LIMIT 15;
```

* See the start of the total exec time

```sql
SELECT stats_reset FROM pg_stat_statements_info; 
```

* manually reset the stats using:

```sql
SELECT pg_stat_statements_reset();
```

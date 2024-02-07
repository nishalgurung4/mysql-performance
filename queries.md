# List of queries:

## To identify most expensive queries

```sql
SELECT
* FROM events_statements_summary_by_digest ORDER BY sum_timer_wait DESC limit 10 \G
```

```sql
SELECT (100 * SUM_TIMER_WAIT / sum(SUM_TIMER_WAIT) OVER ()) AS  percent, SUM_TIMER_WAIT AS total, COUNT_STAR AS calls, AVG_TIMER_WAIT AS mean, substring(DIGEST_TEXT, 1, 75) FROM performance_schema.events_statements_summary_by_digest ORDER BY SUM_TIMER_WAIT DESC LIMIT 10;
```

> [Read More](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-statement-summary-tables.html)

## To check I/O operations metrics

```sql
SELECT * FROM performance_schema.table_io_waits_summary_by_table WHERE
object_name = "city"\G
```

> [Read More](https://dev.mysql.com/doc/mysql-perfschema-excerpt/8.3/en/performance-schema-table-io-waits-summary-by-table-table.html)

## `Explain` for Query Optimization

```sql
EXPLAIN ANALYZE SELECT * FROM world.city;
```

## Test expression index

```sql
USE adbms;
```

```sql
ALTER TABLE test_estimates ADD INDEX idx(val);
```

```sql
CREATE TABLE test_estimates (id INT AUTO_INCREMENT PRIMARY KEY, val INT, val2 INT);
```

```python
queries = [
("INSERT INTO `test_estimates` (`val`, `val2`) VALUES (?, ?)",)
]
for x in range(0, 1000000):
    for query in queries:
        sql = query[0]
        result = session.run_sql(sql, (x,x,))
```

```sql
EXPLAIN ANALYZE SELECT * FROM test_estimates WHERE 2 * val < 3;
```

```sql
ALTER TABLE test_estimates ADD INDEX idx_fun((2 * val));
```

## Example of UUID and AUTO_INCREMENT

```sql
CREATE TABLE users (
  id    int unsigned NOT NULL AUTO_INCREMENT,
  name  varchar(64) NOT NULL DEFAULT '',
  email varchar(64) NOT NULL DEFAULT '',
  dob   date DEFAULT NULL,
  PRIMARY KEY (id),
  UNIQUE KEY email (email)
) ENGINE=InnoDB;
```

```python
import time
queries = [
    ("INSERT INTO `users` (`name`, `email`, `dob`) VALUES (?, ?, ?)", ["John Doe", "john@gmail.com", "1991-02-01"])
]
start_time = time.time()
for x in range(0, 100000):
  for query in queries:
    sql = query[0]
    param = query[1]
    result = session.run_sql(sql, (param[0], param[1] + str(x), param[2],))
print("--- %s seconds ---" % (time.time() - start_time))
```

```sql
CREATE TABLE users_uuid (
  uuid  varchar(36) NOT NULL,
  name  varchar(64) NOT NULL DEFAULT '',
  email varchar(64) NOT NULL DEFAULT '',
  dob   date DEFAULT NULL,
  PRIMARY KEY (uuid),
  UNIQUE KEY email (email)
) ENGINE=InnoDB;
```

```python
import time, uuid
queries = [
    ("INSERT INTO `users_uuid` (`uuid`,`name`, `email`, `dob`) VALUES (?,?, ?, ?)", ["John Doe", "john@gmail.com", "1991-02-01"])
]
start_time = time.time()
for x in range(0, 100000):
  for query in queries:
    sql = query[0]
    param = query[1]
    result = session.run_sql(sql, (str(uuid.uuid4()), param[0], param[1] + str(x), param[2],))
print("--- %s seconds ---" % (time.time() - start_time))
```

### To check composite key

```python
import time
query = ("SELECT count(*) FROM users WHERE `name` = ? AND `dob` = ?", ['John Doe', "1992-01-01"])
nb_of_requests = 300
start_time = time.time()
for x in range(0, nb_of_requests):
  sql = query[0]
  param = query[1]
  result = session.run_sql(sql, (param[0], param[1],))
total_time = time.time() - start_time
qps = nb_of_requests / total_time
print("- Total time: %s seconds -" % total_time)
print("--- %s QPS ---" % qps)
```

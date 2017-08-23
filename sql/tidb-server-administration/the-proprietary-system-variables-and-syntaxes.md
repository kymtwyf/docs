---
Title: specific system variables and syntax used in TiDB
Category: compatibility
---

# The Proprietary System Variables and Syntaxes in TiDB
On the basis of MySQL variables and syntaxes, TiDB has defined some specific system variables and syntaxes to optimize performance.

## System Variable

| Name                                                                        | Default Value  | Scope           |
|:----------------------------------------------------------------------------|:---------------|:----------------|
| [`tidb_distsql_scan_concurrency`](#tidb_distsql_scan_concurrency)           | 10             | SESSION, GLOBAL |
| [`tidb_index_lookup_size`](#tidb_index_lookup_size)                         | 20000          | SESSION, GLOBAL |
| [`tidb_index_lookup_concurrency`](#tidb_index_lookup_concurrency)           | 4              | SESSION, GLOBAL |
| [`tidb_index_serial_scan_concurrency`](#tidb_index_serial_scan_concurrency) | 1              | SESSION, GLOBAL |

Variables can be set with the `SET` statement, for example:

```sql
set @@tidb_distsql_scan_concurrency = 10;
set @@global.tidb_distsql_scan_concurrency = 10; -- set global variable
set @@session.tidb_distsql_scan_concurrency = 10; -- set session variable
```

- <span id="tidb_distsql_scan_concurrency">`tidb_distsql_scan_concurrency`</span>

This variable is used to set the concurrency of the `scan` operation. Use a bigger value in OLAP scenarios, and a smaller value in OLTP scenarios. For OLAP scenarios, the maximum value cannot exceed the number of CPU cores of all the TiKV nodes.

- <span id="tidb_index_lookup_size">`tidb_index_lookup_size`</span>

This variable is used to set the batch size of index lookup operation. Use a bigger value in OLAP scenarios, and a smaller value in OLTP scenarios.

- <span id="tidb_index_lookup_concurrency">`tidb_index_lookup_concurrency`</span>

This variable is used to set the concurrency of the `index lookup` operation. Use a bigger value in OLAP scenarios, and a smaller value in OLTP scenarios.

- <span id="tidb_index_serial_scan_concurrency">`tidb_index_serial_scan_concurrency`</span>

This variable is used to set the concurrency of the `serial scan` operation. Use a bigger value in OLAP scenarios, and a smaller value in OLTP scenarios.

## Optimizer Hint

On the basis of MySQL’s [Optimizer Hint](https://dev.mysql.com/doc/refman/5.7/en/optimizer-hints.html) Syntax, TiDB adds some proprietary `Hint` syntaxes. When using the `Hint` syntax, the TiDB optimizer will try to use the specific algorithm, which performs better than the default algorithm in some scenarios.

The `Hint` syntax is included in comments like `/*+ xxx */`, and in MySQL client versions earlier than 5.7.7, the comment is removed by default. If you want to use the `Hint` syntax in these earlier versions, add the `--comments` option when starting the client. For example: `mysql -h 127.0.0.1 -P 4000 -uroot --comments`.

- `TIDB_SMJ(t1, t2)`

This variable is used to remind the optimizer to use the `Sort Merge Join` algorithm. This algorithm takes up less memory, but takes longer to execute. It is recommended if the data size is too large, or there’s insufficient system memory.

Usage:
```sql
SELECT /*+ TIDB_SMJ(t1, t2) */ * from t1，t2 where t1.id = t2.id;
```

- `TIDB_INLJ(t1, t2)`

This variable is used to remind the optimizer to use the `Index Nested Loop Join` algorithm. In some scenarios, this algorithm runs faster and takes up fewer system resources, but may be slower and takes up more system resources in some other scenarios. You can try to use this algorithm in scenarios where the result-set is less than 10,000 rows after the outer table is filtered by the WHERE condition. The parameter in `TIDB_INLJ()` is the candidate table for the driving table (external table) when generating the query plan. That means, `TIDB_INLJ (t1)` will only consider using t1 as the driving table to create a query plan.

Usage:
```sql
SELECT /*+ TIDB_INLJ(t1, t2) */ * from t1，t2 where t1.id = t2.id;
```

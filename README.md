# VastDB Query Engine ADBC Driver

## Overview

The VastDB ADBC Driver provides access to the VastDB Query Engine.

For more details about the VAST Query Engine, see [this whitepaper](https://kb.vastdata.com/documentation/docs/vast-query-engine).

For more details about the VAST Database, see [this whitepaper](https://vastdata.com/whitepaper/#TheVASTDataBase).

## Requirements

- Linux / Mac client with network access to the VAST Cluster
- [Virtual IP pool configured with DNS service](https://support.vastdata.com/s/topic/0TOV40000000FThOAM/configuring-network-access-v50)
- [S3 access & secret keys on the VAST cluster](https://support.vastdata.com/s/article/UUID-4d2e7e23-b2fb-7900-d98f-96c31a499626)
- [Tabular identity policy with the proper permissions](https://support.vastdata.com/s/article/UUID-14322b60-d6a2-89ac-3df0-3dfbb6974182)


## Installation

The VastDB ADBC Driver is shipped as a standalone library.

## Usage

Example of using the VastDB ADBC Driver with the Python `adbc-driver-manager`.

```python
import adbc_driver_manager

with adbc_driver_manager.dbapi.connect(
    driver="<path/to/driver>",
    db_kwargs={
        "vast.db.endpoint": "<vast_cluster_endpoint>",
        "vast.db.access_key": "<aws_access_key>",
        "vast.db.secret_key": "<aws_secret_key>",
    }
) as conn:
    # Use the database connection
    pass
```

See [Options](#options) for the full list of database, connection, and statement options.

## Options

### Database options

Set these on `AdbcDatabase` before creating a connection. All three are required.

| Option | Values | Default | Description |
|--------|--------|---------|-------------|
| `vast.db.endpoint` | URL string | — | VAST cluster endpoint URL. Required. |
| `vast.db.access_key` | string | — | AWS access key. Required. |
| `vast.db.secret_key` | string | — | AWS secret key. Required. |

Standard ADBC database options are not supported.

### Connection options

#### Standard ADBC options

| Option | Values | Default | Description |
|--------|--------|---------|-------------|
| `adbc.connection.autocommit` | `"true"` or `"false"` | `"true"` | Controls transaction autocommit mode. Can be set at connection creation or via `AdbcConnectionSetOption` before the connection is used (before creating statements, calling `get_objects`, or `get_table_schema`). Cannot be changed after the connection is used. |

#### Custom options

| Option | Values | Default | Description |
|--------|--------|---------|-------------|
| `vast.db.end_user` | string | — | End-user impersonation. Set at connection creation. For more details see [User Impersonation](https://kb.vastdata.com/documentation/docs/user-impersonation). |
| `vast.db.variable.<name>` | string, integer, or double | — | Define a user variable accessible in queries, e.g. `SELECT getVariable('x')`. Any valid DuckDB user variable name is valid. Set after connection init. |
| `vast.db.setting.<name>` | string, integer, or double | — | Set a VAST session setting that influences query execution (see supported settings below). Set after connection init. |

User variables and session settings cannot be passed at connection creation; set them with `AdbcConnectionSetOption` after the connection is initialized.

#### Supported VAST session settings

- `vector_search_skip_recent_non_indexed`
- `vector_search_min_prob`
- `vector_search_max_prob`
- `vector_search_pruning_distance_ratio`
- `vector_search_min_full_clusters_after_filtering`
- `vector_search_defer_projection`

### Statement options

#### Standard ADBC options

Used with `AdbcStatementExecuteUpdate` for bulk data ingest (see [Data ingestion](#data-ingestion)).

| Option | Values | Default | Description |
|--------|--------|---------|-------------|
| `adbc.ingest.mode` | `adbc.ingest.mode.append` | — | Ingest mode. Only append is supported. Required for ingest. |
| `adbc.ingest.target_catalog` | `vastdb` | — | Target catalog. Only `vastdb` is accepted when set. |
| `adbc.ingest.target_db_schema` | schema path | — | Fully qualified schema path (e.g. `"bucket/schema"`). Required for ingest. |
| `adbc.ingest.target_table` | table name | — | Target table name. Required for ingest. |

Other standard ingest options are not supported.

## Data ingestion

The driver supports ADBC bulk ingest via `AdbcStatementExecuteUpdate`, following the standard ADBC ingest option keys listed above.

**Supported data binding**

- `AdbcStatementBind` — bind a single Arrow `RecordBatch`. This is the supported way to provide ingest data.
- `AdbcStatementBindStream` — **not supported**.

**Requirements**

1. Set all required ingest options (`adbc.ingest.mode`, `adbc.ingest.target_db_schema`, `adbc.ingest.target_table`).
2. Bind a `RecordBatch` with `AdbcStatementBind`.
3. An SQL statement must **not** be set on the statement.
4. Call `AdbcStatementExecuteUpdate`.

Ingest respects the connection's transaction mode: use `commit` / `rollback` when autocommit is disabled.

Example using the Python `adbc-driver-manager`:

```python
import adbc_driver_manager
import pyarrow as pa

bucket = "my-bucket"
schema_name = "s"
table_name = "t"

schema = pa.schema([("a", pa.int32())])
batch = pa.record_batch([pa.array([42], type=pa.int32())], schema=schema)

with adbc_driver_manager.dbapi.connect(
    driver="<path/to/driver>",
    db_kwargs={
        "vast.db.endpoint": "<vast_cluster_endpoint>",
        "vast.db.access_key": "<aws_access_key>",
        "vast.db.secret_key": "<aws_secret_key>",
    },
) as conn:
    with adbc_driver_manager.AdbcStatement(conn.adbc_connection) as stmt:
        stmt.set_options(
            **{
                "adbc.ingest.mode": "adbc.ingest.mode.append",
                "adbc.ingest.target_catalog": "vastdb",
                "adbc.ingest.target_db_schema": f"{bucket}/{schema_name}",
                "adbc.ingest.target_table": table_name,
            }
        )
        stmt.bind(batch)
        rows_affected = stmt.execute_update()
```

## Logging

- **Default logging level**: `INFO`.
- To increase verbosity, set the environment variable `RUST_LOG` to values like `DEBUG` or `TRACE`.

### Log Location

Logs are written to the following directory on Linux systems:

`~/.local/share/VastDbDriver/vastdb_driver.log`

Or, on Mac to:

`~/Library/Logs/VastDbDruver/vastdb_driver.log`

- Home directory can be changed by setting `VAST_ADBC_LOG_DIR`
- Console logs can be enabled by settings `VAST_ADBC_STDOUT=1`
- Logs are rotated daily or when the file reaches 100 MB.
- The system keeps up to 10 log files at a time.

## Known Limitations

- **AdbcConnectionGetObjects**: Supports only the `vastdb` catalog and exact filter for `db_schema`
- **ExecutePartitions** Not supported.
- **AdbcStatementPrepare**: Prepare should be done using SQL prepare statements through execute.
- **StatementExecuteSchema**, **AdbcStatementGetParameterSchema** and **AdbcStatementSetSubstraitPlan**: Not supported.
- **AdbcStatementBindStream**: Not supported. Ingest accepts data only via `AdbcStatementBind` with a single `RecordBatch`.
- **Ingest modes**: Only `adbc.ingest.mode.append` is supported (`create`, `create_append`, and `replace` are not).

## Support

For detailed documentation, FAQs, and troubleshooting guides, refer to the official resources or contact the VastDB support team.

--- 

*This driver is compatible with the ADBC interface, and thus integrates seamlessly into existing ADBC workflows, with noted exceptions.*

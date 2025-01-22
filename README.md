# VastDB Query Engine ADBC Driver

## Overview

The VastDB ADBC Driver provides access to the VastDB Query Engine.

At present, the Query Engine supports query operations for selection and filtering.

For more details about the VAST Database, see [this whitepaper](https://vastdata.com/whitepaper/#TheVASTDataBase).

## Requirements

- Linux client with network access to the VAST Cluster
- [Virtual IP pool configured with DNS service](https://support.vastdata.com/s/topic/0TOV40000000FThOAM/configuring-network-access-v50)
- [S3 access & secret keys on the VAST cluster](https://support.vastdata.com/s/article/UUID-4d2e7e23-b2fb-7900-d98f-96c31a499626)
- [Tabular identity policy with the proper permissions](https://support.vastdata.com/s/article/UUID-14322b60-d6a2-89ac-3df0-3dfbb6974182)


## Installation

The VastDB ADBC Driver is shipped as a standalone library.

## Usage

Supply the `AdbcDatabase` with the following options

- `vast.db.endpoint` - VAST cluster endpoint
- `vast.db.access_key` - AWS access key
- `vast.db.secret_key` - AWS secret access key

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

## Logging

- **Default logging level**: `INFO`.
- To increase verbosity, set the environment variable `RUST_LOG` to values like `DEBUG` or `TRACE`.

### Log Location

Logs are written to the following directory on Linux systems:

`~/.local/share/VastDbDriver/vastdb_driver.log`

- Logs are rotated daily or when the file reaches 100 MB.
- The system keeps up to 10 log files at a time.

## Known Limitations

- **AdbcConnectionGetObjects**: Supports only the `vastdb` catalog and exact filter for `db_schema`
- **GetStatistics**: Not supported.
- **ExecutePartitions** Not supported.
- **AdbcStatementPrepare**: Prepare should be done using SQL prepare statements through execute.
- **StatementExecuteSchema**, **AdbcStatementGetParameterSchema** and **AdbcStatementSetSubstraitPlan**: Not supported.

## Support

For detailed documentation, FAQs, and troubleshooting guides, refer to the official resources or contact the VastDB support team.

--- 

*This driver is compatible with the ADBC interface, and thus integrates seamlessly into existing ADBC workflows, with noted exceptions.*

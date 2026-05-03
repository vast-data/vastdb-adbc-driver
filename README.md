# VAST Database QueryEngine ADBC Driver

## Overview

The VAST Database QueryEngine ADBC Driver provides an [ADBC (Arrow Database Connectivity)](https://arrow.apache.org/adbc/current/index.html) compliant interface for seamless integration with the VAST Database QueryEngine. For comprehensive technical specifications, please consult the [ADBC format specification](https://arrow.apache.org/adbc/current/format/specification.html).

## Requirements

* A Linux client with network connectivity to the VAST Cluster.
* [S3 access and secret keys configured on the VAST Cluster](https://kb.vastdata.com/documentation/docs/overview-of-vast-cluster-s3-implementation).
* [A Virtual IP (VIP) pool configured with a corresponding DNS service](https://kb.vastdata.com/documentation/docs/configuring-network-access).
* [Appropriate permission settings for database access](https://kb.vastdata.com/documentation/docs/managing-permissions-for-accessing-vast-tabular-databases).

## Installation

**Via PyPI (Recommended for Python Environments)**  
Install the driver directly from the Python Package Index:
```bash
pip install adbc-driver-vastdb
```

**Via GitHub Releases**  
Pre-compiled binaries are available on the [GitHub Releases page](https://github.com/vast-data/vastdb-adbc-driver/releases).

## Basic Usage

The driver can be instantiated using standard ADBC driver managers across supported languages. Below is an example using the Python standard `adbc_driver_manager`. 

To initialize the connection, provide the driver path and the required database credentials. If the driver was installed via PyPI, you can utilize the `get_driver_path()` function to locate the binary dynamically. For advanced usage and implementations in other programming languages, please refer to the official ADBC documentation.

### Python DB-API Connection Example

```python
import adbc_driver_manager.dbapi
import adbc_driver_vastdb

with adbc_driver_manager.dbapi.connect(
    driver=adbc_driver_vastdb.get_driver_path(), 
    db_kwargs={
        "vast.db.endpoint": "http://<vast-endpoint>",
        "vast.db.access_key": "<aws_access_key>",
        "vast.db.secret_key": "<aws_secret_key>"
    }
) as conn:
    # Utilize the Python DB-API connection
    pass 
```

## Configuration Options

The driver supports various VAST-specific configuration parameters applicable at the Database, Connection, and Statement levels.

### Database Options
Provided during initialization (e.g., via `db_kwargs`):
* `vast.db.endpoint`: The fully qualified URL of the VAST server.
* `vast.db.access_key`: The AWS access key ID for authentication.
* `vast.db.secret_key`: The AWS secret access key for authentication.

### Connection Options
Provided during connection instantiation (e.g., via `conn_kwargs`):
* `adbc.connection.autocommit`: Toggles autocommit behavior (`"true"` or `"false"`).
  * IMPORTANT - if using via the `adbc-driver-manager` use `autocommit` parameter of the connect function, as it has precedence.
* `vast.db.end_user`: Specifies the end-user identity for impersonation purposes.

### Environment Variables
* `VAST_ADBC_LOG_DIR`: Specifies a custom directory for log files, overriding the default path.
* `VAST_ADBC_STDOUT`: When set, redirects all driver logging to standard output rather than the file system.

## Logging

The driver implements a daily rotating log mechanism to record operations and errors. It retains a maximum of 10 log files, rotating them either daily or when a file size exceeds 100 MB. 

If the `VAST_ADBC_LOG_DIR` environment variable is omitted, logs are stored in the following OS-specific default directory:
* **Linux:** `~/.local/share/VastAdbcDriver/vastdb_driver.log`

## Support

For comprehensive documentation, frequently asked questions, and troubleshooting assistance, please refer to the official VAST Data resources or contact the VastDB support team.

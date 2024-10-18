# ODBC Configuration for Trovares Desktop

This document provides detailed instructions on how to configure ODBC support for the Trovares Desktop application.

## Overview

ODBC (Open Database Connectivity) support enables the application to interact with various database systems via ODBC drivers. This guide will help you set up ODBC in the Docker environment provided by Trovares Desktop.

## Preparing ODBC Configuration Files

To enable ODBC support, you need to set up your `odbc.ini` and `odbcinst.ini` files along with the necessary ODBC drivers.
You will need to mount all the ODBC files and drivers to `/odbc` in the container with a Docker volume mount.

In our example we are going to assume you are connecting to MariaDB which has a ODBC driver called `libmaodbc.so`.
This driver requires a library file called `libmariadb.so.3` to work.

Follow these steps to prepare and configure the files:

1. **ODBC Files and Drivers:**
   - Prepare your `odbc.ini` and `odbcinst.ini` files. These will contain the configurations needed to establish database connections.
   - These should be set up according to the specific requirements of the database you are connecting to.
   - Obtain the ODBC drivers compatible with Debian 11 and ensure they are compatible with the architecture of your machine (e.g., x86_64, aarch64, or ppc64le).
   - The obc.ini would contain a Data Source Name(DSN) and some connection information such as the driver, server, port, user, password, etc:
     ```ini
     [MariaDB-Server]
     Description = MariaDB server
     Driver = MariaDB
     Server = 192.168.50.173
     Port = 3306
     Option = 3
     ```
     There shouldn't be much changes to your typical `odbc.ini`.
   - The `obcinst.ini` would contain the driver information, but needs to make sure it points to the correct driver location that is mounted in the container:
     ```ini
     [MariaDB]
     Description = ODBC Driver for MariaDB
     Driver = /odbc/libmaodbc.so
     FileUsage = 1
     ```
     The `Driver = MariaDB` line in odbc.ini indicates use the driver above. Note the driver in the container is mounted to `/odbc`.

2. **Double Check the Driver Path:**
   - In your `odbcinst.ini`, ensure that the driver paths are correctly pointed to within the Docker container. For example:
     ```ini
     Driver = /odbc/libmaodbc.so
     ```
   - This path refers to where the driver will be located inside the container, not on your host machine.

3. **Setting Up the Docker Volume:**
   - Place the `odbc.ini`, `odbcinst.ini`, and driver files in a directory on your host machine, for example, `./odbc`.
   - Assuming MariaDB, you would have placed `libmaodbc.so` and `libmariadb.so.3` into `./odbc`.
   - The library search path for this mount point is `/odbc`. So ensure all .so files are in this top-level directory.
   - Ensure the Docker Compose file mounts this directory to `/odbc` in the container:
     ```yaml
     - ./odbc:/odbc
     ```

## Running with ODBC Support

With the configuration files and drivers set up, include the volume mount in your Docker Compose setup to enable ODBC support.
Uncomment and configure the line:

```yaml
services:
  backend:
    volumes:
      - ./odbc:/odbc
```

## Testing the Configuration

To verify that ODBC is set up correctly:

- Start the Trovares Desktop.
- Add a connection in the Profile tab to the database using the DSNs or driver specified in your `odbc.ini` or `odbcinst.ini`.
- An example for MariaDB would be: `Driver={MariaDB};Server=127.0.0.1;Port=3306;Database=test;Uid=test;Pwd=foo;`
- In the Upload tab perform a test query to ensure the connection is successfully established.

## Troubleshooting

If you encounter issues with ODBC connectivity:

- Read the error messages returned by upload. They will usually indicate the general issue.
- Ensure that the file permissions for `odbc.ini`, `odbcinst.ini`, and the driver files allow them to be read by the Docker container.
- Check that the driver paths in `odbcinst.ini` accurately reflect their mounted location in the Docker container.
- Make sure you are using the libraries for Debian 11 and the system architecture.
- Review the Docker container logs for any ODBC-related errors:

```bash
    docker logs backend
```

## Advanced Troubleshooting

If the driver still isn't being found by the ODBC Manager, it may mean that all the library dependencies aren't being resolved.

To validate the library should be working, you need to inspect the library in the container:

First find the backend container ID:

```bash
    docker container ls
```

Connect to the container:

```bash
    sudo docker exec -it YOUR_CONTAINER_ID /bin/bash
```

Inspect the library:

```bash
    ldd /odbc/driver.so
```

If the ldd command fails, it means your driver is the wrong architecture.
Otherwise, the ldd command will list the dependencies that will look something like this:

```bash
    linux-vdso.so.1 (0x00007ffd5f7b4000)
    libmariadb.so.3 => not found
    libodbcinst.so.2 => /usr/lib/x86_64-linux-gnu/libodbcinst.so.2 (0x00007f87e6bf9000)
```

Notice, the missing library.
In this case we didn't put all the libraries the driver needed in the odbc directory.

For further details and support, you can always refer back to the main [README file](./README.md) or contact support@trovares.com

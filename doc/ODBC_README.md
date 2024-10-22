# ODBC Configuration for Trovares Desktop

This document provides detailed instructions on how to configure ODBC (Open Database Connectivity) support for the Trovares Desktop application.
ODBC support enables the desktop to interact with various database systems via ODBC drivers.

## Preparing ODBC Configuration Files

To enable ODBC support, the `odbc.ini` and `odbcinst.ini` files need to be set up along with the necessary ODBC drivers.
The ODBC files and drivers need to be placed in a directory which will be volume mounted to one of the desktop Docker containers.

As an example this document describes setting up MariaDB.  Other databases are set up similarly.  MariaDB has an ODBC driver called `libmaodbc.so`.
This driver requires a library file called `libmariadb.so.3` to work.

Follow these steps to prepare and configure the files:

1. **Set up the ODBC Files and Drivers:**
   - Prepare the `odbc.ini` and `odbcinst.ini` files. These files contain the configurations needed to establish database connections and should be set up according to the specific requirements of the database being connecting to.

   - Obtain the ODBC drivers compatible with Debian 11 and ensure they are compatible with the architecture of the host machine (e.g., x86_64, aarch64, or ppc64le).

   - The drivers and initialization files are mounted to `/odbc` in the Docker container.

   - The `obc.ini` file contains a Data Source Name (DSN) and some connection information such as the driver, server, port, user, password, etc:
     ```ini
     [MariaDB-Server]
     Description = MariaDB server
     Driver = MariaDB
     Server = 192.168.50.173
     Port = 3306
     Option = 3
     ```
     This should be similar to a typical `odbc.ini`.

   - The `obcinst.ini` file contains driver information. It must point to the location where the driver is mounted in the container.
     ```ini
     [MariaDB]
     Description = ODBC Driver for MariaDB
     Driver = /odbc/libmaodbc.so
     FileUsage = 1
     ```
     The `Driver = MariaDB` line in the `odbc.ini` file indicates use the driver above.

2. **Double Check the Driver Path:**
   - In the `odbcinst.ini` file, ensure that the driver paths are correctly pointed to within the Docker container. For example:
     ```ini
     Driver = /odbc/libmaodbc.so
     ```

   - This path refers to where the driver will be located inside the container, not on the host machine.

3. **Set up the Docker Volume:**
   - Place the `odbc.ini`, `odbcinst.ini`, and driver files in a directory on the host machine. For example, `./odbc`.

   - For MariaDB, the driver files `libmaodbc.so` and `libmariadb.so.3` would be placed into `./odbc` along with the initialization files.

   - All the files must be directly in the directory on the host machine and not in subdirectories.

   - Set the environment variable `TD_ODBC_PATH` to the directory where the driver files are placed.  We suggest setting the environment variable in a `.env` file.  An example entry in a `.env` file is:
     ```bash
     TD_ODBC_PATH=./odbc
     ```

## Testing the Configuration

To verify that ODBC is set up correctly:

- Start the Trovares Desktop.
- Add a connection in the Profile tab to the database using the DSNs or driver specified in `odbc.ini` or `odbcinst.ini`.
- An example for MariaDB would be: `Driver={MariaDB};Server=127.0.0.1;Port=3306;Database=test;Uid=test;Pwd=foo;`
- In the Upload tab, perform a test query to ensure the connection is successfully established.

## Troubleshooting

If you encounter issues with ODBC connectivity:

- Read the error messages returned by upload. They will usually indicate the general issue.
- Ensure that the file permissions for `odbc.ini`, `odbcinst.ini`, and the driver files allow them to be read by the Docker container.
- Check that the driver paths in `odbcinst.ini` accurately reflect their mounted location in the Docker container.
- Make sure the driver files are for Debian 11 and the host system's architecture.
- Review the Docker container logs for any ODBC-related errors:

```bash
    docker logs backend
```

## Advanced Troubleshooting

If the driver still isn't being found by the ODBC Manager, it may mean that all the library dependencies aren't being resolved. To determine what is going wrong, inspect the library in the container.

 1. Find the backend container ID:
    ```bash
        docker container ls
    ```

 1. Connect to the container:
    ```bash
        sudo docker exec -it YOUR_CONTAINER_ID /bin/bash
    ```

 1. Inspect the library:
    ```bash
        ldd /odbc/driver.so
    ```

 1. If the ldd command fails, it means the driver is for the wrong architecture.
    Otherwise, the ldd command will list the dependencies that will look something like this:
    ```bash
        linux-vdso.so.1 (0x00007ffd5f7b4000)
        libmariadb.so.3 => not found
        libodbcinst.so.2 => /usr/lib/x86_64-linux-gnu/libodbcinst.so.2 (0x00007f87e6bf9000)
    ```
    Notice, the missing library. In this case all the needed libraries weren't put in the `./odbc` directory.

For further details and support, refer back to the main [README](../README.md) or contact [support@trovares.com](mailto:support@trovares.com).

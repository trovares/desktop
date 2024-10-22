# Trovares Desktop

**Unveiling the power of data through intuitive and dynamic visualizations.**

Trovares Desktop is an innovative web application designed to revolutionize the way organizations visualize and interact with complex datasets. The application offers a comprehensive suite of tools that facilitate the interactive exploration of data through graph-based visualizations. It integrates powerful features such as dynamic data loading, customizable views, and an intuitive user interface, all tailored to enhance the user experience in data analysis and visualization. The desktop is designed to be responsive and user-friendly, ensuring seamless navigation and an enriching data interaction experience.

For organizations managing large and intricate datasets, particularly in critical sectors like defense cybersecurity, finance, and healthcare, Trovares Desktop stands as a pivotal tool. It delivers high-performance analytics and insightful visual representations, enabling users to uncover hidden patterns, relationships, and trends in data. This level of insight is crucial for decision-making and strategic planning in high-stakes environments. The application's capacity to handle massive datasets efficiently makes it an invaluable asset for organizations seeking to transform their data into actionable intelligence, thereby fostering informed decisions and enhancing operational effectiveness.

Trovares Desktop is a web application for driving property graph workloads in the [Trovares xGT server](https://docs.trovares.com).

## Quick Installation

Perform these steps to install and run Trovares Desktop and Trovares xGT on a server.

 1. Make sure Docker is running.  You may need to start (or install) a [Docker Desktop](https://docs.docker.com/desktop/) or [Docker Engine](https://docs.docker.com/engine/).  To verify that Docker is working, run the following command.  You should see information about the Docker environment.
    ```bash
    docker info
    ```

 1. Copy the `docker-compose.yml` file from this repo to your server or laptop.

 1. If installing on IBM Power Series, create a `.env` file in the same directory as the `docker-compose.yml` file containing this line:

    ```bash
    export TD_MONGODB_IMAGE=ibmcom/mongodb-ppc64le
    ```

 1. Start Trovares Desktop and Trovares xGT:
    ```bash
    docker compose up -d
    ```

 1. Aim a browser to `localhost` on the system running this Docker application and log in to the dekstop.

## Installation

Trovares Desktop uses Docker Compose with these Docker images:
 - [trovares/xgt](https://hub.docker.com/r/trovares/xgt)
 - [trovares/desktop_frontend](https://hub.docker.com/r/trovares/desktop_frontend)
 - [trovares/desktop_backend](https://hub.docker.com/r/trovares/desktop_backend)
 - [mongo](https://hub.docker.com/_/mongo)

Trovares Desktop can be run using either Docker Desktop or Docker Engine only.  Further references to Docker Engine in the Installation section refer to a Docker Engine install without Docker Desktop.

### Configuration for the xGT Server

The frontend, backend, and database containers must be run on the same host.  However, the xGT server can be run in the following ways:
 - In a Docker container as part of the Compose project.
 - In an isolated Docker container (separate from the Compose project) on the same host as the desktop.
 - Installed from an RPM on the same host as the desktop.
 - On a different host than the desktop, either in a Docker container or installed from an RPM.

#### xGT as Part of the Compose Project

No extra setup is necessary when xGT is run as part of the compose project.

The hostname to use when logging into Trovares Desktop is either `xgt`, the name of the service for the xGT server in the docker-compose.yml file, or the host's external IP.

#### xGT in an Isolated Container on the Desktop Host

Here is an example of starting the xGT server in an isolated container:
```bash
docker run --name xgt -d -p 4367:4367 -v /host/data/dir:/data -v /host/conf/dir:/conf -v /host/log/dir:/var/log/xgtd -v /host/ssl/dir:/ssl trovares/xgt
```
This command exposes port 4367 to the host.  The xGT server listens on port 4367.  Exposing this port is required for Trovares Desktop to communicate with the isolated container.  The command also volume maps a data directory, a config directory, a log directory, and a directory containing ssl certs for encrypting traffic to the xGT server.  Change the command to map only the directories you need and point to the correct host directories.  See the [documentation for running xGT in a Docker container](https://docs.trovares.com/using_docker_image/index.html) for more details.

Comment out or delete the xgt section in the docker-compose.yml file.

Use the host's external IP as the hostname when logging into Trovares Desktop.

Another option for the login hostname is to use either `localhost` or `host.docker.internal`, but further configuration is required if using Docker Engine to run Trovares Desktop.  In that case, add the following lines in the backend section of the docker-compose.yml file:
```yaml
    extra_hosts:
      - "host.docker.internal:host-gateway"
```
Docker Desktop automatically provides the translation of "host.docker.internal" to the gateway IP of the default bridge network.  These lines add the translation in Docker Engine.  The desktop translates "localhost" to "host.docker.internal" to provide a shorter more commonly understood hostname.

#### xGT Installed from an RPM on the Desktop Host

The xGT server configuration variable `system.hostname` must be set appropriately when connecting Trovares Desktop to an RPM installed xGT.  See [the xGT configuration documentation](https://docs.trovares.com/sysadmin_guide/configuration.html) for more details.  One option is to set "system.hostname" to the host's external IP.  If access on 127.0.0.1 is desired in addition to access via the host's external IP, set "system.hostname" to "0.0.0.0".

To setup access via 127.0.0.1 but no external access, the setup is slightly more complicated.  If using Docker Desktop to run Trovares Desktop, use the default value of "localhost" for "system.hostname".  If using Docker Engine to run Trovares Desktop, "system.hostname" must be set to the gateway IP of Docker's default bridge network.  The gateway IP is almost always "172.17.0.1".  To verify the gateway IP, do
```bash
docker network inspect bridge
```
Look for a section like this
```
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
```
The gateway IP is the value for "Gateway".

After setting the value for "system.hostname" and starting the xGT server, comment out or delete the xgt section in the docker-compose.yml file.

Use the host's external IP as the hostname when logging into Trovares Desktop.

If the xGT server is configured for access via 127.0.0.1, another option for the login hostname is to use either `localhost` or `host.docker.internal`.  However, further configuration is required if using Docker Engine to run Trovares Desktop.  In that case, uncomment the following lines in the backend section of the docker-compose.yml file:
```yaml
    extra_hosts:
      - "host.docker.internal:host-gateway"
```
Docker Desktop automatically provides the translation of "host.docker.internal" to the gateway IP of the default bridge network.  These lines add the translation in Docker Engine.  The desktop translates "localhost" to "host.docker.internal" to provide a shorter more commonly understood hostname.

#### xGT on a Different Host

The xGT server can be running on the other host either in a Docker container or installed from an RPM.

Comment out or delete the xgt section in the docker-compose.yml file.

Use the IP of the host where the xGT server is running as the hostname when logging into Trovares Desktop.

### Environment Variables

There are a number of environment variables that configure the desktop and xGT server.  Variables that start with TD_ configure the desktop, while variables that start with XGT_ configure the server.  We suggest putting definitions of the environment variables in a .env file in the same directory as the docker-compose.yml file.  That way they will be available for all Docker Compose commands.

Here is an example .env file that sets up running the webserver using SSL.

```dotenv
TD_SSL_PUBLIC_CERT=/directory/to/ssl/td-cert.pem
TD_SSL_PRIVATE_KEY=/directory/to/ssl/td-private-key.pem
```

The configurable environment variables are:

|Variable            |Volume Mapped|Description|
|--------------------|-|-----------|
|TD_PORT             | |alternative port for the http web server|
|TD_SSL_PORT         | |alternative port for the https web server|
|TD_SSL_PUBLIC_CERT  |Y|path to certificate on host to setup an https web server|
|TD_SSL_PRIVATE_KEY  |Y|path to private key on host to setup an https web server|
|TD_MONGO_URI        | |location of the database used by the desktop|
|TD_DEFAULT_XGT_HOST | |default login host for the desktop|
|TD_DEFAULT_XGT_PORT | |default login port for the desktop|
|TD_ODBC_PATH        | |path to ODBC drivers for the connector|
|TD_MONGODB_IMAGE    | |used to specify the mongodb image for Power10 installs|
|XGT_SSL_SERVER_CERT |Y|path to chain file on host for the xGT server’s certificate|
|XGT_SERVER_CN       | |common name on the xGT server’s certificate|
|XGT_DATA_PATH       |Y|path to the data directory on host for the xGT server|
|XGT_AUTH_TYPES      | |sets xGT server authentication types available in desktop|

The variables that are volume mapped map point to a file or directory on the host that gets mapped to an expected location in the containers.

### Instructions

 1. Copy the `docker-compose.yml` file from this repo to your server or laptop.

 1. If the machine you want to run on doesn’t have internet access, download all the Docker images on a machine connected to the internet.  The machine you download on must have the same architecture as the machine you want to run on.

    1. Download the Docker images.
       ```bash
       docker pull mongo
       docker pull trovares/xgt
       docker pull trovares/desktop_frontend
       docker pull trovares/desktop_backend
       ```

    1. Save the Docker images to file.  Make sure to use the `<image>:<tag>` format to specify the image for the save command.  Otherwise you might have to manually add tags when loading later.
       ```bash
       docker save -o mongo.tar mongo:latest
       docker save -o xgt.tar trovares/xgt:latest
       docker save -o desktop_frontend.tar trovares/desktop_frontend:latest
       docker save -o desktop_backend.tar trovares/desktop_backend:latest
       ```

    1. Copy the Docker image tar files to the machine they are to be installed on.

    1. Load the Docker images:
       ```bash
       docker load -i mongo.tar
       docker load -i xgt.tar
       docker load -i desktop_frontend.tar
       docker load -i desktop_backend.tar
       ```

 1. If running the xGT server as part of the Compose project, setup a data directory using the environment variable XGT_DATA_PATH.  The default is /tmp if XGT_DATA_PATH is not set.  For example:
    ```dotenv
    XGT_DATA_PATH=/path/to/data/dir
    ```

 1. (Optional) Setup using SSL to connect from the desktop to the xGT server.  The xGT server must also be configured to use SSL.  (See https://docs.trovares.com/sysadmin_guide/configuration.html.)  Set the environment variables XGT_SSL_SERVER_CERT and XGT_SERVER_CN.  For example:
    ```dotenv
    XGT_SSL_SERVER_CERT=/directory/to/ssl/ca-chain.cert.pem
    XGT_SERVER_CN=’TrovaresServer'
    ```

 1. (Optional) Setup certificates for connecting from the browser to the desktop over https.  Set the environment variables TD_SSL_PUBLIC_CERT and TD_SSL_PRIVATE_KEY to the certificate and private key for the web server.  For example:
    ```dotenv
    TD_SSL_PUBLIC_CERT=/directory/to/ssl/td-public.pem
    TD_SSL_PRIVATE_KEY=/directory/to/ssl/td-private.pem
    ```

 1. (Optional) Set a default host and port for when a user first logs into the desktop using the environment variables TD_DEFAULT_XGT_HOST and TD_DEFAULT_XGT_PORT.  These only affect the first time a user logs in as the host and port from the last login are cached in their browser after that.  If not set the defaults are “xgt” (for connecting to the xGT Docker image) and 4367 (default xGT server port).  For example:
    ```dotenv
    TD_DEFAULT_XGT_HOST=192.168.1.1
    TD_DEFAULT_XGT_PORT=4368
    ```

 1. (Optional) Select the xGT server authentication types available to desktop users using the environment variable XGT_AUTH_TYPES.  The supported types are 'BasicAuth', which uses a username and password, and 'PKIAuth'.  The default is to support both types.  The value of XGT_AUTH_TYPES must be a string representing a JSON list of the selected types.  This example allows only username / password authentication:
    ```dotenv
    XGT_AUTH_TYPES="['BasicAuth']"
    ```

 1. If upgrading, pull the latest versions of the Docker containers:
    ```bash
    docker compose pull
    ```

 1. Start Trovares Desktop:
    ```bash
    docker compose up -d
    ```

 1. Aim a browser to the system running this Docker application and log in to the desktop.

## Database Connectivity

Trovares Desktop supports loading data from a database.  Refer to the [ODBC documentation](doc/ODBC_README.md) to connect to a database.

The supported databases are:
 - MongoDB
 - Oracle
 - SAP: ASE and IQ
 - Snowflake
 - Generic ODBC: Databricks, DB2, MySQL, and MariaDB

## License

By downloading, installing or using any of these images you agree to the [license agreement](https://docs.trovares.com/EULA/xGT_License_for_Containers.pdf) for this software.

As with all Docker images, these likely also contain other software which may be under other licenses (such as Bash, etc from the base distribution, along with any direct or indirect dependencies of the primary software being contained).

As for any pre-built image usage, it is the image user's responsibility to ensure that any use of this image complies with any relevant licenses for all software contained within.

# Trovares Desktop

The Trovares Desktop is part of the Trovares Graph application.

Trovares Graph includes both the Trovares Desktop and the [Trovares xGT server](https://docs.trovares.com).  The Desktop is a web application for driving property graph workloads in the
xGT server.


## Quick Installation

Perform these steps to install and run the Trovares Graph application on a server.

 1. Copy the `docker-compose.yml` file from this repo to your server or laptop.
 1. Pull copies of Docker containers to your environment:
    ```bash
    docker compose pull
    ```

 1. Start Trovares Graph:
    ```bash
    docker compose up -d
    ```

 1. Aim a browser to `localhost:80` on the system running this Docker application.

## Installation

Trovares Desktop uses Docker Compose with these Docker images:
 - [trovares/xgt](https://hub.docker.com/r/trovares/xgt)
 - [trovares/desktop_frontend](https://hub.docker.com/r/trovares/desktop_frontend)
 - [trovares/desktop_backend](https://hub.docker.com/r/trovares/desktop_backend)
 - [mongo](https://hub.docker.com/_/mongo)

The frontend, backend, and database containers must be run on the same host.  However, the desktop supports several setups that change how the xGT server is run and where it is located:
 - Run the xGT server in a Docker container as part of the Compose project.
 - Run the xGT server in an isolated Docker container (separate from the Compose project) on the same host as the desktop.
 - Run the xGT server in either a Docker container or as a direct install on a different host than the desktop.


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
|XGT_SSL_SERVER_CERT |Y|path to chain file on host for the xGT server’s certificate|
|XGT_SERVER_CN       | |common name on the xGT server’s certificate|
|XGT_DATA_PATH       |Y|path to the data directory on host for the xGT server|

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
    1. Save the Docker images to file.  Make sure to use the <image_name>:<tag> format to specify the image for the save command.  Otherwise you might have to manually add tags when loading later.
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

 1. If not running the xGT server as part of the Compose project, comment out or delete the xgt section in the docker-compose.yml file.  Make sure the external xGT server is running.

 1. If running the xGT server on the same host as Trovares Desktop and using Docker Engine, uncomment the following lines in the backend section of the docker-compose.yml file.
    ```yaml
    extra_hosts:
      - "host.docker.internal:host-gateway"
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

 1. (Optional) Set a default host and port for when a user first logs into the desktop using the environment variables TD_DEFAULT_XGT_HOST and TD_DEFAULT_XGT_PORT.  These only affect the first time a user logs in as the host and port from the last login are cached in their browser after that.  If not set the defaults are “xgt” (for connecting to docker xgt image) and 4367 (default xGT server port).  For example:
    ```dotenv
    TD_DEFAULT_XGT_HOST=192.168.1.1
    TD_DEFAULT_XGT_PORT=4368
    ```

 1. Start Trovares Graph:
    ```bash
    docker compose up -d
    ```

 1. Aim a browser to the system running this Docker application.  If required, login using authentication information for the xGT server.

## License

By downloading, installing or using any of these images you agree to the [license agreement](https://docs.trovares.com/EULA/xGT_License_for_Containers.pdf) for this software.

As with all Docker images, these likely also contain other software which may be under other licenses (such as Bash, etc from the base distribution, along with any direct or indirect dependencies of the primary software being contained).

As for any pre-built image usage, it is the image user's responsibility to ensure that any use of this image complies with any relevant licenses for all software contained within.

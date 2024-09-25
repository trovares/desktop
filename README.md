# Trovares Desktop

The Trovares Desktop is part of the Trovares Graph application.

The Desktop is a web application for driving property graph workloads in the
[Trovares xGT server](https://docs.trovares.com).


# Installation

Perform these steps to install the Trovares graph application on a server.

  - Copy the `docker-compose.yml` file from this repo to your server or laptop.
  - Pull copies of docker containers to your environment:

```bash
docker compose pull
```

  - You can start a Trovares Application, which includes both the Desktop and the Trovares graph server,  with this command:
```bash
docker compose up -d
```

  - You can then aim a browser to `localhost:80` on the system running this docker application.

## License

By downloading, installing or using any of these images you agree to the [license agreement](https://docs.trovares.com/EULA/xGT_License_for_Containers.pdf) for this software.

As with all Docker images, these likely also contain other software which may be under other licenses (such as Bash, etc from the base distribution, along with any direct or indirect dependencies of the primary software being contained).

As for any pre-built image usage, it is the image user's responsibility to ensure that any use of this image complies with any relevant licenses for all software contained within.

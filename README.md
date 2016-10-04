# Automatic vhosts

This repository defines a convention for a basic web development environment
that uses Docker. The environment supports running multiple projects
simultaneously and in isolation.

## Convention

 - A bridge network called `auto` is used to collect all user-facing web
   services.
   
   Containers added to this network usually have multiple interfaces, with one
   connected to the `auto` network, and others used to reach internal services.

 - A container running the [jwilder/nginx-proxy] image connected to the `auto`
   network, and with port 80 exposed on the host machine.

   This container must have access to the host Docker socket, mounted as
   `/tmp/docker.sock`, in order to discover other containers.
   
   The container runs an Nginx server as a reverse proxy, and will
   automatically reconfigure itself with virtual hosts according to other
   running containers that have an environment variable `VIRTUAL_HOST` set.
   These virtual hosts should all be subdomains of the `docker` top-level
   domain.
   
   See the [jwilder/nginx-proxy] documentation for details.

 - A wildcard domain `*.docker` which resolves to the Docker host.

   This ensures that the virtual hosts are reachable through the proxy.

 [jwilder/nginx-proxy]: https://hub.docker.com/r/jwilder/nginx-proxy/

## Example setup

This repository contains an example setup that implements the above convention
using [Docker Compose] and the [andyshinn/dnsmasq] image.

To run this setup:

 * Install [Docker for Mac], [Docker for Windows] or just plain [Docker] if you
   are already on a Linux host.

   Check the Docker daemon is running and available by running:
   `docker version`. Also make sure the `docker-compose` tool is available, or
   you may have chosen an installation method that packages Docker Compose
   separately.

 * See the `docker-compose.yml` file, and adjust it to your needs.
 
   The section 'Customizing' below contains some tips for useful adjustments.

 * Create the `auto` bridge network:

   `docker network create auto`

 * Launch the containers:

   `docker-compose up -d`

   The `-d` option will run the containers in the background once they have
   been created.

 * Configure your OS to use the nameserver: `127.0.0.1`

 [Docker Compose]: https://docs.docker.com/compose/
 [andyshinn/dnsmasq]: https://hub.docker.com/r/andyshinn/dnsmasq/
 [Docker for Mac]: https://docs.docker.com/docker-for-mac/
 [Docker for Windows]: https://docs.docker.com/docker-for-windows/
 [Docker]: https://docs.docker.com/engine/installation/linux/

## Customizing

 - The default DNS settings use Google DNS as the forward DNS server. Edit the
   `docker-compose.yml` file to adjust it if needed.

 - On Linux or macOS (with e.g. [homebrew]), it's also possible to run
   `dnsmasq` on the host. Simply comment or remove the entire `dnsmasq`
   container in `docker-compose.yml`, install the `dnsmasq` package on your
   host, and use a configuration like:

```
listen-address=127.0.0.1
address=/docker/127.0.0.1
server=8.8.8.8
```

 - In general, `dnsmasq` can be replaced with any DNS server, as long as
   there's a wildcard domain `*.docker` that resolves to localhost.

 - More advanced setups may run the Docker daemon somewhere other than
   `localhost`. If this is the case, you will need to change the `--address`
   option to the `dnsmasq` container to point to the correct IP address.

 [homebrew]: http://brew.sh/

## Configuring projects

Projects should start user-facing web services with an additional interface
connected to the `auto` network, and with an environment variable
`VIRTUAL_HOST` set to a hostname in the `docker` top-level domain.

An example project `docker-compose.yml`:

```yaml
version: '2'

networks:
  auto:
    external: true

services:
  whoami:
    image: jwilder/whoami
    networks:
      - default
      - auto
    environment:
      - VIRTUAL_HOST=whoami.docker
```

Save this file as `docker-composer.yml` in a new directory, then run
`docker-compose up` in this directory. You should now be able to visit
[whoami.docker](http://whoami.docker/).

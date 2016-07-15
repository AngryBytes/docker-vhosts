# Automatic vhosts example

An example setup for creating automatic vhosts in Docker during development.

## Usage

 * Install [Docker for Mac], [Docker for Windows] or just plain [Docker] if you
   are already on a Linux host.

 * Create the `auto` docker network:

   `docker network create auto`

 * Launch the containers:

   `docker-compose up -d`

 * Configure your OS to use the nameserver: `127.0.0.1`

 [Docker for Mac]: https://docs.docker.com/docker-for-mac/
 [Docker for Windows]: https://docs.docker.com/docker-for-windows/
 [Docker]: https://docs.docker.com/engine/installation/linux/

## Configuring projects

Projects should start webserver containers connected to the `auto` network, and
with an environment variable `VIRTUAL_HOST` in the `.docker` TLD.

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

## Notes

Builds on [jwilder/nginx-proxy] and [andyshinn/dnsmasq]. See the
`docker-compose.yml` file, and adjust to your needs.

Notably, the default DNS settings use Google DNS as the forward DNS server,
which can be adjusted.

 [jwilder/nginx-proxy]: https://hub.docker.com/r/jwilder/nginx-proxy/
 [andyshinn/dnsmasq]: https://hub.docker.com/r/andyshinn/dnsmasq/

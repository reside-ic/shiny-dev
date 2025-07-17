# shiny-dev

This is a temporary repo designed to give an easy-to-understand deployment of the apache / haproxy / shiny stack; it does not include configurable applications or anything like that.

## Usage

Start the system

```
docker compose up
```

Bring everything down

```
docker compose down
```

Interact with the `twinkle` cli

```
./twinkle <args...>
```

## The components

### apache

Running an unmodified httpd container (previously was 2.4, we'll update once we know this works). The configuration ([`apache/httpd.conf`](httpd/httpd.conf)) and certificates (`apache/ssl`) will be read-only mounted into the container. You need to fetch the ssl key and certificate, run `./apache/configure_ssl` to do this (only needs to be done if they change or if the `ssl` directory is deleted)

### haproxy

An unmodified version of the latest `haproxy`, with a configuration [`haproxy/haproxy.cfg`](haproxy/haproxy.cfg); currently a single shiny worker assumed.

### shiny

An unmodified version the twinkle shiny container, with the configuration and site configuration mounted in.  Apps and their libraries are stored in a persistant docker volume which is also mounted in.

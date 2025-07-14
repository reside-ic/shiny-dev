# shiny-dev

This is a temporary repo designed to give an easy-to-understand deployment of the apache / haproxy / shiny stack; it does not include configurable applications or anything like that.

## The components

### apache

Running an unmodified httpd container (previously was 2.4, we'll update once we know this works). The configuration ([`apache/httpd.conf`](httpd/httpd.conf)) and certificates (`apache/ssl`) will be read-only mounted into the container. You need to fetch the ssl key and certificate, run `./apache/configure_ssl` to do this (only needs to be done if they change or if the `ssl` directory is deleted)

### haproxy

An unmodified version of the latest `haproxy`, with a configuration [`haproxy/haproxy.cfg`](haproxy/haproxy.cfg); currently a single shiny worker assumed.

### shiny

An unmodified version of the official shiny container

### apps

Not yet configured, soon to come from a volume.

Some applications (copied over from the original deployment are in `apps`). These will want to be in a volume; run `./apps/create_volume` to copy them into the volume


## Bringing the bits up

```
docker compose up
```

Teardown

```
docker compose down
```

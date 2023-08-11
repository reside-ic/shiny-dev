# shiny-dev

This is a temporary repo designed to give an easy-to-understand deployment of the apache / haproxy / shiny stack; it does not include configurable applications or anything like that. See it online at https://shiny-dev.dide.ic.ac.uk (DIDE network only).

## The components

Some of these are docker images, and they are not set to pull, so you will want to rebuild. All build very quickly.

### apache

Running an unmodified httpd container (previously was 2.4, we'll update once we know this works). The configuration ([`apache/httpd.conf`](httpd/httpd.conf)) and certificates (`apache/ssl`) will be read-only mounted into the container. You need to fetch the ssl key and certificate, run `./apache/configure_ssl` to do this (only needs to be done if they change or if the `ssl` directory is deleted)

### haproxy

Build the image with `./haproxy/build` which builds `mrcide/haproxy` with a configuration that can be seen in [`haproxy/haproxy.cfg`](haproxy/haproxy.cfg) and some utilities which enable some degree of dynamic scaling of shiny servers.

### shiny

A lightly modified version of the official shiny container; the original version was more extensively modified

### apps

Some applications (copied over from the original deployment are in `apps`). These will want to be in a volume; run `./apps/create_volume` to copy them into the volume

### Summary

```
./apache/configure_ssl
./haproxy/build
./shiny/build
./apps/create_volume
```

## Bringing the bits up

```
docker network create twinkle 2> /dev/null || /bin/true
docker volume create shiny_logs
docker run -d --name haproxy --network twinkle mrcide/haproxy:dev
docker run -d --name apache --network twinkle \
  -p 80:80 \
  -p 443:443 \
  -p 9000:9000 \
  -v "${PWD}/apache/httpd.conf:/usr/local/apache2/conf/httpd.conf:ro" \
  -v "${PWD}/apache/auth:/usr/local/apache2/conf/auth:ro" \
  -v "${PWD}/apache/ssl:/usr/local/apache2/conf/ssl:ro" \
  httpd:2.4
docker run -d --name shiny-1 --network=twinkle \
  -v twinkle_apps:/shiny/apps \
  -v twinkle_logs:/shiny/logs \
  -p 3838:3838 \
  mrcide/shiny-server:dev
docker exec haproxy update_shiny_servers shiny 1
```

Teardown

```
docker rm -f haproxy apache shiny-1
docker network rm twinkle
```

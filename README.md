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

## The configuration

* `site.yml`: the main configuration describing all applications. This will be discussed below
* `shiny-server.conf`: the configuration for the shiny server itself. Very little here should need changing, and this could possibly be baked into the twinkle image
* `haproxy.cfg`: configuration for the load-balancer
* `httpd.conf`: configuration for apache. This needs to match the configuration of the site itself that is serving the applications

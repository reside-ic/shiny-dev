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

```bash
docker network create twinkle 2> /dev/null || /bin/true
docker volume create shiny_logs
docker run -d --name haproxy --network twinkle mrcide/haproxy:dev
docker run -d --name apache --network twinkle \
  -p 80:80 \
  -p 443:443 \
  -p 9005:9005 \
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

```bash
docker rm -f haproxy apache shiny-1
docker network rm twinkle
```

## Running in Kubernetes

The following is guide to run the shiny server in kubernetes. They are based on running a Kind cluster. The configuration may
nee to be adjusted for other k8s clusters.

### Prerequisites

A single node k8s cluster is needed. To setup a k8s cluster follow the guide [here](https://mrc-ide.myjetbrains.com/youtrack/articles/RESIDE-A-31/Setting-up-Kubernetes-k8s-Cluster).

Run the following commands  from the root of the repository

1. `./shiny/build`
2. For KIND cluster load image: `kind load docker-image mrcide/shiny-server:dev`
3. Create /shiny/logs and /shiny/apps directories in k8s node or run `./k8s/configure_ssl`. For kind exec into node via `docker exec -it kind-control-plane sh`
 and then `mkdir -p /shiny/logs /shiny/apps` to create the directories.
3.`k apply -k k8s/overlays/production` for production and `k apply -k k8s/overlays/staging` for staging.
4.`k8s/shiny-sync`

Great the shiny server is running and can be seen on the External IP of the ingress-nginx-controller LoadBalancer service.

#### Teardown

`k delete -k k8s/overlays/production` for production or `k delete -k k8s/overlays/staging` for staging

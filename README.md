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

A production kubernetes cluster using k3s is needed to be setup first. To setup a k8s cluster follow the guide [here](https://mrc-ide.myjetbrains.com/youtrack/articles/RESIDE-A-31/Setting-up-Kubernetes-k8s-Cluster). Note: If using dev cluster, the storageClassName: local-path is unavailable and you will need to provision own storage class and persistant volume. 

Run `start-k8s-shiny <env>` to run the shint server in k8s. 

The server can be seen running on the External IP of the ingress-nginx-controller LoadBalancer service.
Note: `kubectl get svc -n ingress-nginx ingress-nginx-controller` command to get external IP.

#### Teardown

Run the following: 

1. `kubectl delete -k k8s/overlays/<env>`. Replace <env> with staging or production. 
2. `kubectl  delete ns twinkle` to remove namespace. 

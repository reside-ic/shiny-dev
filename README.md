# shiny-dev

This is a temporary repo designed to give an easy-to-understand deployment of the apache / haproxy / shiny stack; it does not include configurable applications or anything like that. See it online at https://shiny-dev.dide.ic.ac.uk (DIDE network only).


## Running in Kubernetes

The following is guide to run the shiny server in kubernetes. They are based on running a Kind cluster. The configuration may
nee to be adjusted for other k8s clusters.

### Prerequisites

A production kubernetes cluster using k3s is needed to be setup first. To setup a k8s cluster follow the guide [here](https://mrc-ide.myjetbrains.com/youtrack/articles/RESIDE-A-31/Setting-up-Kubernetes-k8s-Cluster). Note: If using dev cluster (KIND), the storageClassName: local-path & longhorn is unavailable and you will need to provision own storage class and persistant volume. 

Run `start-k8s-shiny <env>` to run the shint server in k8s.


#### Teardown

Run the following: 

1. `kubectl delete -k k8s/overlays/<env>`. Replace <env> with testing or production. 
2. `kubectl  delete ns twinkle` to remove namespace. 

# shiny-dev

This is a temporary repo designed to give an easy-to-understand deployment of the apache / haproxy / shiny stack; it does not include configurable applications or anything like that. See it online at https://shiny-dev.dide.ic.ac.uk (DIDE network only).

## Running in Kubernetes

### Prerequisites

A k8s kubernetes cluster using k3s is needed to be setup first. To setup a k8s cluster follow the guide [here](https://mrc-ide.myjetbrains.com/youtrack/articles/RESIDE-A-31/Setting-up-Kubernetes-k8s-Cluster).

Run `./start-k8s-shiny <env>` to run the shiny server in k8s.

Note: If on testing enviroment the app will launch on the IP addr of the result of the following command: 
`kubectl -n ingress-nginx get svc ingress-nginx-controller -o=jsonpath='{.status.loadBalancer.ingress[0].ip}'`

#### Teardown

Run the following: 

1. `kubectl delete -k k8s/overlays/<env>`. Replace <env> with testing or production. 
2. `kubectl  delete ns twinkle` to remove namespace. 

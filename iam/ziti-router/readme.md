https://github.com/openziti/helm-charts/blob/main/charts/ziti-router/README.md#ziti-router
ziti-router

Version: 1.0.0 Type: application AppVersion: 1.0.0

Host an OpenZiti router in Kubernetes

Add the OpenZiti Charts Repo to Helm

helm repo add openziti https://docs.openziti.io/helm-charts/
Minimal Installation

After adding the charts repo to Helm, then you may install the chart in the same cluster where the controller is running by using the cluster-internal service of the control plane endpoint. 

This default values used in this minimal approach is suitable for a Kubernetes distribution 

like K3S or Minikube that configures pass-through TLS for Service resources of type LoadBalancer.

configuring router with k3 with service resource = loadbalancer
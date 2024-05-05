#https://openziti.io/helm-charts/

Chart Highlights

Charts for Workloads// used for k8s and non k8s workloads

These charts help cluster workloads access or provide a Ziti service.

ziti-host: Ziti tunnel pod for hosting services (ingress only)
ziti-edge-tunnel: Ziti tunnel daemonset for accessing services (intercept node egress)


Charts for Self-Hosting Ziti

ziti-controller
ziti-router
ziti-console


Charts that Deploy a Ziti-enabled Application

httpbin: Ziti fork of the REST testing server
prometheus: Ziti fork of Prometheus
reflect: A Ziti original. This app echoes the bytes it receives and is useful for testing Ziti.
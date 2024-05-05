#https://openziti.io/helm-charts/charts/ziti-controller/

Requirements

Repository	Name	Version
https://charts.jetstack.io	cert-manager	~1.11.0
https://charts.jetstack.io	trust-manager	~0.4.0
https://kubernetes.github.io/ingress-nginx/	ingress-nginx	~4.5.2
Note that ingress-nginx is not strictly required, but the chart is parameterized to allow for conveniently declaring pass-through TLS.

NOTE: CHANGES REQUIRED TO MODIFY NGINX TO ENABLE SSL PASSTHROUGH 

You must patch the ingress-nginx deployment to enable the SSL passthrough option.



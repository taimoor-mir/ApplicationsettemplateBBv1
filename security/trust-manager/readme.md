https://cert-manager.io/docs/trust/trust-manager/

trust-manager is a small Kubernetes operator which aims to help reduce the overhead of managing TLS trust bundles in your clusters.

It adds the Bundle custom Kubernetes resource (CRD) which can 
read input from various sources 

and 

combine the resultant certificates 

into a bundle ready to be used by your applications.

trust-manager ensures that it's both quick and easy to keep your trusted certificates up-to-date and enables cluster administrators to easily automate providing a secure bundle without having to worry about rebuilding containers to update trust stores.

It's designed to complement cert-manager and works well when consuming CA certificates from a cert-manager Issuer or ClusterIssuer but can be used entirely independently of cert-manager if needed.


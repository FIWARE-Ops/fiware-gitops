# Routes & Certs

In order to support different hosts and getting them secured automatically, we are using additional operators on our cluster:

* [External-DNS](https://github.com/openshift/external-dns-operator) - allows to automatically create subdomains for hosted-zones
* [Cert-Manager](https://cert-manager.io/v0.15-docs/installation/openshift/) - automatically provide and update ssl-certificates using [Lets encrypt](https://letsencrypt.org/)
* [Cert-Utils](https://github.com/redhat-cop/cert-utils-operator) - automatically inject certificates into openshift' route-objects

## External-DNS

The operator is installed via OperatorHub:


1. Search for the operator:
![OperatorHub](./external-dns-search.png)

2. Fulfill the documented prerequisites(e.g. create the namespace and roles, according to the documentation)
>:warning: In the current documentation, the operator uses a broken link for installing the extra-roles, use this "https://github.com/openshift/external-dns-operator/blob/main/config/rbac/extra-roles.yaml" instead or replace "main" with the version tag required.

3. Install the operator:
![OperatorHub](./external-dns-install.png)

4. Create a user for the operator. For AWS, follow the [official documentation](https://github.com/openshift/external-dns-operator/blob/main/docs/usage.md#aws)

5. Create an external-dns resource, handling route-objects. See [external-dns/dns-resources/aws-routes-fiware-dev.yaml](../aws/external-dns/dns-resources/aws-routes-fiware-dev.yaml) as an example.

## Cert-Manager

The operator is installed via OperatorHub:

1. Search for the operator:
![OperatorHub](./certmanager-search.png)

2. Install: 
![OperatorHub](./certmanager-install.png)
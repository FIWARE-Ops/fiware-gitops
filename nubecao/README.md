# Kubernetes cluster 'nubecao'

## Prerequisites

* A running Kubernetes cluster (e.g. k3s, kubeadm, EKS, GKE, AKS)
* kubectl access with cluster-admin permissions
* helm cli installed and running
* An Ingress controller or LoadBalancer support
* DNS records for external access

## Basic components

### Argo CD

Argo CD is used to deploy and manage all platform components via GitOps.

Argo CD can be installed via the following helm command after adjusting the argocd.yaml file, especially for the login mechanism. The currently used one, is based on github users of a specific organisation. The configuration for general SSO usage can be found [here](https://argo-cd.readthedocs.io/en/stable/operator-manual/user-management/#2-configure-argo-cd-for-sso) with specific guides for github and others available in the internet.

```
helm repo add argo https://argoproj.github.io/argo-helm
helm upgrade -i argo-cd argo/argo-cd --version 9.3.4 -f argocd.yaml -n argo-cd
```

Once installed, log into argocd and import a root application to bootstrap all deployments of the cluster. If the login fails due to missing certificates, use a temporary login using kubectl as described [here](https://argo-cd.readthedocs.io/en/stable/getting_started/#port-forwarding).

```
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: ''
spec:
  project: default
  source:
    repoURL: https://github.com/FIWARE-Ops/fiware-gitops.git
    path: nubecao/applications
    targetRevision: HEAD
    directory:
		recurse: true
		jsonnet: {}
  destination:
    server: https://kubernetes.default.svc
    namespace: argocd
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

### Cert-Manager

cert-manager is required to issue and manage TLS certificates for platform services and will be installed via Argo-CD. It is recommended to check the configuration in ```nubecao/cert-manager/templates/letsencyrpt-prod.yaml``` and adjust to the operator.

### Database Operators

Databases are provisioned using the Zalando postgres-operator and the mariadb-operator.

### Sealed secrets

Secrets are loaded into the cluster by using Bitnami's Sealed-Secret container and must be recreated according to this [guide](https://github.com/bitnami-labs/sealed-secrets).
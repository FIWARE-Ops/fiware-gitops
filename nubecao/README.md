# Kubernetes cluster 'nubecao'

## Argo CD

Root Application to bootstrap all other applications:

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
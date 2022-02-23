# ArgoCD Application resources

As an alternative to the ArgoCD UI, [applications](https://argo-cd.readthedocs.io/en/stable/core_concepts/) also can be deployed via the [argocd-cli](https://argo-cd.readthedocs.io/en/stable/cli_installation/#installation)
This folder contains the app-definitions, that are equal to the UI-based creation described in the [main-documentation](../../README.md).

Following steps are required to apply them:

### 1. Login argocd-cli

```shell
   argocd login --sso  $(kubectl get routes -n argocd -o json | jq  -r '.items[0].spec.host')
```

### 2. Create the applications

To have everything running smooth, you should create the applications in the same order as described by the UI-based installation. Use the files in the current folder for that.
```shell
    argocd app create -f <APPLICATION_YAML>
```

After all apps are deployed, check there state via:
```shell
  argocd app list --grpc-web
  
  #expected output:
  NAME             CLUSTER                                     NAMESPACE       PROJECT  STATUS  HEALTH   SYNCPOLICY  CONDITIONS  REPO                                         PATH                 TARGET
  fiware-mongo-db  https://api.fiware-dev-aws.fiware.dev:6443  fiware          default  Synced  Healthy  Auto-Prune  <none>      https://github.com/FIWARE-Ops/fiware-gitops  aws/fiware/mongodb   HEAD
  fiware-orion-ld  https://api.fiware-dev-aws.fiware.dev:6443  fiware          default  Synced  Healthy  Auto-Prune  <none>      https://github.com/FIWARE-Ops/fiware-gitops  aws/fiware/orion-ld  HEAD
  fiware-secrets   https://api.fiware-dev-aws.fiware.dev:6443  fiware          default  Synced  Healthy  Auto-Prune  <none>      https://github.com/FIWARE-Ops/fiware-gitops  aws/fiware/secrets   HEAD
  namespaces       https://api.fiware-dev-aws.fiware.dev:6443                  default  Synced  Healthy  Auto        <none>      https://github.com/FIWARE-Ops/fiware-gitops  aws/namespaces       HEAD
  sealed-secrets   https://api.fiware-dev-aws.fiware.dev:6443  sealed-secrets  default  Synced  Healthy  Auto-Prune  <none>      https://github.com/FIWARE-Ops/fiware-gitops  aws/sealed-secrets   HEAD

```
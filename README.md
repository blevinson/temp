## Install ArgoCD:
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Use the following configurations



https://162.0.213.73:8080

https://argocd.k8s.infra.wiser.com/applications/bootstrap-infra-argo

https://github.com/blevinson/temp

path: deploy
Revision: main
namespace: argo
Application Name: bootstrap-infra-argo
prune: false
self heal: false
automated: true
auto-sync: true

Not using: https://raw.githubusercontent.com/argoproj/argo/stable/manifests/install.yaml

Using: https://github.com/argoproj/argo-workflows/blob/master/manifests/quick-start-postgres.yaml

Version 3.4.1: https://github.com/argoproj/argo-workflows/blob/v3.4.1/manifests/quick-start-postgres.yaml

https://argoproj.github.io/argo-workflows/installation/


From Infra's version:


https://github.com/argoproj/argo-workflows

v2.12.7

manifests/namespace-install

Get Authentication: kubectl -n argo exec argo-server-${POD_ID} -- argo auth token

Install Argo Events:
Repo: https://github.com/argoproj/argo-events
Path: manifests/namespace-install

Notes:
1) Figure out how Workflow and Events can both be deployed via namespace-install /
 and work together? Research this. We may have to add them both into our own /
repo?

Combined Install:
Name: bootstrap-infra-argo
URL: https://github.com/blevinson/temp
Path: deploy



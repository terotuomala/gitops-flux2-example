# GitOps workflow example using Flux2

> Inspired by excellent [Flux2 example](https://github.com/fluxcd/flux2-kustomize-helm-example).

<!-- TABLE OF CONTENTS -->
## Table of Contents 
* [Features](#features)
* [Example application](#example-application)
* [Prerequisites](#prerequisites)
* [Usage](#usage)
* [Kustomize configuration](#kustomize-configuration)

<!-- FEATURES -->
## :rocket: Features 
- Local [K3s](https://github.com/rancher/k3s) Cluster using [K3d](https://github.com/rancher/k3d)
- Example application (separate repositories) including [Client](https://github.com/terotuomala/k8s-create-react-app-example) (Create React App) and [REST API](https://github.com/terotuomala/k8s-express-api-example) (Node.js + Express + Redis)
- Application configuration customization using [Kustomize](https://github.com/kubernetes-sigs/kustomize)
- Continuous Delivery with GitOps workflow using [Flux2](https://github.com/fluxcd/flux2)
- Progressive delivery with canary releases using [Flagger](https://github.com/weaveworks/flagger)
- Kubernetes configuration best practises using [Kyverno](https://github.com/kyverno/kyverno)
- Network traffic flow control using [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
- Kubernetes Secrets using [Sealed Secrets](https://github.com/bitnami-labs/sealed-secrets)
- Environment variable loading based on the present working directory using [direnv](https://github.com/direnv/direnv)
- Kubernetes manifest validation using [pre-commit](https://github.com/pre-commit/pre-commit)

<!-- EXAMPLE APPLICATION -->
## :performing_arts: Example Application
The example application consist from Client and REST API with Cache. 

## :card_file_box: Folder structure
The folders are structured based on the [Flux2 example](https://github.com/fluxcd/flux2-kustomize-helm-example).

```
├── infrastructure
│   ├── nginx
│   ├── redis
│   └── sources
└── clusters
    ├── staging
    └── production
```
### Infrastructure
Includes `nginx` and `redis` configurations as well as `Helm Repository` definitions. It also includes example application `Git Repository` definitions ([api.yaml](https://github.com/terotuomala/gitops-flux2-example/blob/main/infrastructure/sources/api.yaml) and [client.yaml](https://github.com/terotuomala/gitops-flux2-example/blob/main/infrastructure/sources/client.yaml))

```
└── infrastructure
    ├── nginx
    │   ├── kustomization.yaml
    │   ├── namespace.yaml
    │   └── release.yaml
    ├── redis
    │   ├── kustomization.yaml
    │   ├── kustomizeconfig.yaml
    │   ├── namespace.yaml
    │   ├── release.yaml
    │   └── values.yaml
    └── sources
        ├── api.yaml
        ├── bitnami.yaml
        ├── client.yaml
        └── kustomization.yaml
```

### Clusters
Includes the Flux configuration per cluster.

```
└── clusters
    ├── production
    │   ├── apps.yaml
    │   └── infrastructure.yaml
    └── staging
        ├── apps.yaml
        └── infrastructure.yaml
```

The `<CLUSTER_ENVIRONMENT>/apps.yaml` defines the path for Kustomize patch which includes the environment specific values. Note that the path refers to different repository. For example the [infrastructure/sources/api.yaml](https://github.com/terotuomala/gitops-flux2-example/blob/main/infrastructure/sources/api.yaml) refers to repository https://github.com/terotuomala/k8s-express-api-example which has the Kustomization files in `k8s/<CLUSTER_ENVIRONMENT>/api` directory.
```
apiVersion: kustomize.toolkit.fluxcd.io/v1beta1
kind: Kustomization
metadata:
  name: api
  namespace: flux-system
spec:
  interval: 1m
  dependsOn:
    - name: infrastructure
  path: "./k8s/staging/api"
  prune: true
  sourceRef:
    kind: GitRepository
    name: api
  validation: client
  healthChecks:
    - kind: Deployment
      name: api
      namespace: api
  timeout: 80s
  ```

<!-- PREREQUISITES -->
## :hammer_and_wrench: Prerequisites
**NB.** The setup is tested on `macOS Big Sur`.

Docker Desktop [installed](https://docs.docker.com/install/)
```sh
# If you don't want to sign up in order to download Docker
# use the following command to download the installer directly
$ curl -s https://download.docker.com/mac/stable/Docker.dmg
```

Kubectl (at least version 1.18) [installed](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
```sh
$ brew install kubernetes-cli
```

Flux CLI [installed](https://toolkit.fluxcd.io/guides/installation/)
```sh
$ brew install fluxcd/tap/flux
```

K3d (at least version v3.4.0) [installed](https://github.com/rancher/k3d)
```sh
$ brew install k3d
```

<!-- USAGE -->
## :keyboard: Usage
Create staging k3s cluster using k3d:
```sh
$ k3d cluster create gitops-example-staging --agents 2 --update-default-kubeconfig
```
Make sure your KUBECONFIG points to staging k3s cluster context:
```sh
$ kubectl get nodes
```
Verify that staging k3s cluster satisfies the prerequisites:
```sh
$ flux check --pre
```
Install Flux and configure it to manage itself from a Git repository:
```sh
$ flux bootstrap github \
    --context=k3d-gitops-example-staging \
    --owner=${GITHUB_USER} \
    --repository=${GITHUB_REPO} \
    --branch=main \
    --personal \
    --path=clusters/staging
```
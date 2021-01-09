# GitOps workflow example using Flux2

<!-- TABLE OF CONTENTS -->
## Table of Contents 
* [Features](#rocket-features)
* [Example application](#performing_arts-example-application)
* [Flux directory structure](#card_file_box-flux-directory-structure)
  * [Infrastructure](#infrastructure)
  * [Clusters](#clusters)
* [Prerequisites](#hammer_and_wrench-prerequisites)
* [Usage](#keyboard-usage)
* [Kustomize configuration](#kustomize-configuration)
* [Thanks](#pray-thanks)

<!-- FEATURES -->
## :rocket: Features
- Local [K3s](https://github.com/rancher/k3s) staging and production clusters  using [K3d](https://github.com/rancher/k3d)
- Example application (separate repositories) including [Client](https://github.com/terotuomala/k8s-create-react-app-example) and [REST API](https://github.com/terotuomala/k8s-express-api-example)
- Application configuration customization using [Kustomize](https://github.com/kubernetes-sigs/kustomize)
- Continuous Delivery with GitOps workflow using [Flux2](https://github.com/fluxcd/flux2)
- Progressive delivery with canary releases using [Flagger](https://github.com/weaveworks/flagger)
- Kubernetes configuration best practises using [Kyverno](https://github.com/kyverno/kyverno)
- Network traffic flow control using [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
- Kubernetes Secrets using [Sealed Secrets](https://github.com/bitnami-labs/sealed-secrets)
- Environment variable loading based on the present working directory using [direnv](https://github.com/direnv/direnv)
- Kubernetes manifest validation using [pre-commit](https://github.com/pre-commit/pre-commit)

<!-- EXMAPLE APPLICATION -->
## :performing_arts: Example Application
The example application consist from [Single-page Application](https://github.com/terotuomala/k8s-create-react-app-example) (Create React App) and [REST API](https://github.com/terotuomala/k8s-express-api-example) with Cache (Node.js + Express.js + Redis)

<!-- FLUX DIRECTORY STRUCTURE -->
## :card_file_box: Flux directory structure
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

The `<CLUSTER_ENVIRONMENT>/apps.yaml` defines the path for Kustomize patch which includes the environment specific values. Note that the `path` refers to a directory which is at different GitHub repository.

```yaml
apiVersion: kustomize.toolkit.fluxcd.io/v1beta1
kind: Kustomization
metadata:
  name: api
  namespace: flux-system
spec:
  interval: 1m
  dependsOn:
    - name: infrastructure
  path: "./k8s/staging" # https://github.com/terotuomala/k8s-express-api-example/tree/main/k8s/staging
.....

---
apiVersion: kustomize.toolkit.fluxcd.io/v1beta1
kind: Kustomization
metadata:
  name: client
  namespace: flux-system
spec:
  interval: 1m
  dependsOn:
    - name: infrastructure
  path: "./k8s/staging" # https://github.com/terotuomala/k8s-create-react-app-example/tree/main/k8s/staging
.....
```

<!-- PREREQUISITES -->
## :hammer_and_wrench: Prerequisites
**NB.** The setup is tested on `macOS Big Sur`.

Docker Desktop [installed](https://hub.docker.com/editions/community/docker-ce-desktop-mac/)
```sh
$ brew install docker
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

Direnv [installed](https://direnv.net/docs/installation.html)
```sh
$ brew install direnv
```

Kubectx [installed](https://github.com/ahmetb/kubectx)
```sh
$ brew install kubectx
```

<!-- USAGE -->
## :keyboard: Usage
Create a local `staging` k3s cluster:
```sh
$ k3d cluster create gitops-example-staging --agents 1 --update-default-kubeconfig
```
Make sure your KUBECONFIG points to staging k3s cluster context:
```sh
$ kubectx k3d-gitops-example-staging
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
Next create local `production` k3s cluster:
```sh
$ k3d cluster create gitops-example-production --agents 1 --update-default-kubeconfig
```
Make sure your KUBECONFIG points to production k3s cluster context:
```sh
$ kubectx k3d-gitops-example-production
```
Verify that production k3s cluster satisfies the prerequisites:
```sh
$ flux check --pre
```
Install Flux and configure it to manage itself from a Git repository:
```sh
$ flux bootstrap github \
    --context=k3d-gitops-example-production \
    --owner=${GITHUB_USER} \
    --repository=${GITHUB_REPO} \
    --branch=main \
    --personal \
    --path=clusters/production
```

<!-- THANKS -->
## :pray: Thanks
Inspired by these excellent projects: 
- https://github.com/fluxcd/flux2-kustomize-helm-example
- https://github.com/onedr0p/home-cluster

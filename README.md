# GitOps workflow example using Flux2
[![Lint](https://github.com/terotuomala/gitops-flux2-example/workflows/Lint/badge.svg)](https://github.com/terotuomala/gitops-flux2-example/actions)

A simple example of managing local K3s clusters (staging and production) including example applications using Flux2, Helm and Kustomize.

<!-- TABLE OF CONTENTS -->
## Table of Contents
* [Features](#rocket-features)
* [Example application](#performing_arts-example-application)
* [Flux directory structure](#card_file_box-flux-directory-structure)
  * [Infrastructure](#infrastructure)
  * [Clusters](#clusters)
* [Prerequisites](#hammer_and_wrench-prerequisites)
* [Usage](#keyboard-usage)
* [Thanks](#pray-thanks)

<!-- FEATURES -->
## :rocket: Features
- Local [K3s](https://github.com/rancher/k3s) staging and production clusters using [K3d](https://github.com/rancher/k3d)
- Example applications (separate repositories) including [Single-page Application](https://github.com/terotuomala/k8s-create-react-app-example) and [REST API](https://github.com/terotuomala/k8s-express-api-example)
- Continuous Delivery with GitOps workflow using [Flux2](https://github.com/fluxcd/flux2)
- Scheduled upgrade check of Flux2 using [Renovate](https://docs.renovatebot.com)
- Policies to secure Kubernetes Pods using [Kyverno](https://github.com/kyverno/kyverno)
- Network traffic flow control using [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
- Environment variable loading using [direnv](https://github.com/direnv/direnv)
- YAML validation using [yamllint](https://github.com/adrienverge/yamllint)

<!-- EXMAPLE APPLICATION -->
## :performing_arts: Example Application
The example applications consist from: 
- [Single-page Application](https://github.com/terotuomala/k8s-create-react-app-example) (Create React App)
- [REST API](https://github.com/terotuomala/k8s-express-api-example) with in-memory data store (Node.js + Express.js + Redis)

The applications are located in separated GitHub repositories and are deployed to the K3s cluster using Flux2.

<!-- FLUX DIRECTORY STRUCTURE -->
## :card_file_box: Flux directory structure
The folders are structured based on the [Flux2 example](https://github.com/fluxcd/flux2-kustomize-helm-example).

```
├── infrastructure
│   ├── calico
│   ├── descheduler
│   ├── kyverno
│   ├── nginx
│   ├── redis
│   └── sources
└── clusters
    ├── staging
    └── production
```
### Infrastructure
Includes `calico`, `descheduler`, `kyverno`, `nginx` and `redis` configurations as well as `Helm Repository` definitions. It also includes example applications `Git Repository` definitions ([api.yaml](https://github.com/terotuomala/gitops-flux2-example/blob/main/infrastructure/sources/api.yaml) and [client.yaml](https://github.com/terotuomala/gitops-flux2-example/blob/main/infrastructure/sources/client.yaml))

```
└── infrastructure
    ├── calico
    │   └── calico.yaml
    ├── descheduler
    │   ├── kustomization.yaml
    │   └── release.yaml
    ├── kyverno
    │   ├── kustomization.yaml
    │   ├── namespace.yaml
    │   └── release.yaml
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
        ├── kubernetes-sigs.yaml
        ├── kustomization.yaml
        └── kyverno.yaml
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

The `clusters/<CLUSTER_ENVIRONMENT>/apps.yaml` defines the path for Kustomize patch which includes the environment specific values. Note that the `path` refers to a directory which is at different GitHub repository.

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

K3d (at least version v4.4.0) [installed](https://github.com/rancher/k3d)
```sh
$ brew install k3d
```

Fork your own copy of this repository to your GitHub account and create a [personal access token](https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token) and export the following variables:
```sh
export GITHUB_TOKEN=<PERSONAL_ACCESS_TOKEN>
export GITHUB_USER=<GITHUB_USERNAME>
export GITHUB_REPO=<GITHUB_REPO_NAME>
```

<!-- USAGE -->
## :keyboard: Usage
Both K3s clusters are using Calico instead of Flannel in order to be able to use NetworkPolicy.

### Local K3s staging cluster
Create the cluster:
```sh
$ k3d cluster create gitops-example-staging \
    --servers 1 \
    --agents 2 \
    --api-port 6550 \
    --port "8080:80@loadbalancer" \
    --k3s-server-arg '--no-deploy=traefik' \
    --k3s-server-arg '--flannel-backend=none' \
    --volume "$(pwd)/infrastructure/calico/calico.yaml:/var/lib/rancher/k3s/server/manifests/calico.yaml" \
    --wait
```
Make sure your KUBECONFIG points to staging k3s cluster context:
```sh
$ kubectl config use-context k3d-gitops-example-staging
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
    --path=clusters/staging \
    --network-policy=false
```
Flux2 is configured to deploy content of the `infrastructure` items using Helm before the application. Verify that the infrastructure Helm releases are synchronized to the cluster:
```sh
$ flux get helmreleases --all-namespaces
```

Verify that the api and client applications are synchronized to the cluster:
```sh
$ flux get kustomizations
```

The example applications should be accessible via Ingress: 
- Single-page Application: `http://localhost:8080`
- REST API: `http://api.localhost:8080`

### (Optional) Local K3s production cluster
Create the cluster:
```sh
$ k3d cluster create gitops-example-production \
    --servers 1 \
    --agents 2 \
    --api-port 6550 \
    --port "8081:80@loadbalancer" \
    --k3s-server-arg '--no-deploy=traefik' \
    --k3s-server-arg '--flannel-backend=none' \
    --volume "$(pwd)/infrastructure/calico/calico.yaml:/var/lib/rancher/k3s/server/manifests/calico.yaml" \
    --wait
```
Make sure your KUBECONFIG points to production k3s cluster context:
```sh
$ kubectl config use-context k3d-gitops-example-production
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
    --path=clusters/production \
    --network-policy=false
```

Verify that the infrastructure Helm releases are synchronized to the cluster:
```sh
$ flux get helmreleases --all-namespaces
```

Verify that the api and client applications are synchronized to the cluster:
```sh
$ flux get kustomizations
```

The example applications should be accessible via Ingress: 
- Single-page Application: `http://localhost:8081`
- REST API: `http://api.localhost:8081`


<!-- THANKS -->
## :pray: Thanks
Inspired by these excellent projects:
- https://github.com/fluxcd/flux2-kustomize-helm-example
- https://github.com/onedr0p/home-cluster

# GitOps-Flux2-Example

An example GitOps workflow using [Flux2](https://github.com/fluxcd/flux2) and [Flagger](https://github.com/weaveworks/flagger).

https://github.com/fluxcd/kustomize-controller

<!-- TABLE OF CONTENTS -->
## Table of Contents :card_file_box: 
* [Features](#features)
* [Kubernetes Objects](#kubernetes-objects)
* [Prerequisites](#prerequisites)
* [Usage](#usage)
* [Kustomize configuration](#kustomize-configuration)

<!-- FEATURES -->
## Features :rocket: 
- Local [K3s](https://github.com/rancher/k3s) Cluster using [K3d](https://github.com/rancher/k3d)
- Example application including Client ([React](https://reactjs.org/)), REST API ([Node.js](https://nodejs.org/en/) + [Express](https://expressjs.com/)) and Cache ([Redis](https://redis.io/))
- Application configuration customization using [Kustomize](https://github.com/kubernetes-sigs/kustomize)
- Continuous Delivery with GitOps workflow using [Flux2](https://github.com/fluxcd/flux2)
- Progressive delivery with canary releases using [Flagger](https://github.com/weaveworks/flagger)
- Kubernetes configuration best practises using [Kyverno](https://github.com/kyverno/kyverno)
- Network traffic flow control using [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
- Kubernetes Secrets using [Sealed Secrets](https://github.com/bitnami-labs/sealed-secrets)
- Environment variable loading based on the present working directory using [direnv](https://github.com/direnv/direnv)
- Kubernetes manifest validation using [pre-commit](https://github.com/pre-commit/pre-commit)

<!-- KUBERNETES OBJECTS -->
## Kubernetes Objects :blue_book:
The following applications and xxx runs in Kubernetes Cluster:

| Type |   | Client   | REST API   | Cache |
|:----------|---|:--------:|:----------:|:-------:|
| Ingress | <img src="https://github.com/kubernetes/community/blob/master/icons/svg/resources/unlabeled/ing.svg" alt="Ingress" title="Ingress resource" width="34,39" height="33" /> | :heavy_check_mark: | :heavy_check_mark: | :x:  |
| Service | <img src="https://github.com/kubernetes/community/blob/master/icons/svg/resources/unlabeled/svc.svg" alt="Service" title="Service resource" width="34,39" height="33" /> | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| Deployment | <img src="https://github.com/kubernetes/community/blob/master/icons/svg/resources/unlabeled/deploy.svg" alt="Deployment" title="Deployment resource" width="34,39" height="33" /> | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| Config Map | <img src="https://github.com/kubernetes/community/blob/master/icons/svg/resources/unlabeled/cm.svg" alt="Config Map" title="Config Map resource" width="34,39" height="33" /> | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| Secret | <img src="https://github.com/kubernetes/community/blob/master/icons/svg/resources/unlabeled/secret.svg" alt="Secret" title="Secret resource" width="34,39" height="33" /> | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| Pod Disruption Budget | <img src="https://github.com/kubernetes/community/blob/master/icons/svg/resources/unlabeled/quota.svg" alt="Pod Disruption Budget" title="Pod Disruption Budget resource" width="34,39" height="33" /> | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| Cluster Policy | <img src="https://github.com/kubernetes/community/blob/master/icons/svg/resources/unlabeled/psp.svg" alt="Cluster Policy" title="Kyverno Cluster Policy resource" width="34,39" height="33" /> | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| Network Policy | <img src="https://github.com/kubernetes/community/blob/master/icons/svg/resources/unlabeled/netpol.svg" alt="Network Policy" title="Network Policy resource" width="34,39" height="33" /> | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |
| Horizontal Pod Autoscaler | <img src="https://github.com/kubernetes/community/blob/master/icons/svg/resources/unlabeled/hpa.svg" alt="Horizontal Pod Autoscaler" title="Horizontal Pod Autoscaler resource" width="34,39" height="33" /> | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: |

## Folder structure
The folders are structured based on 

- **apps** dir contains Helm releases with a custom configuration per cluster
- **infrastructure** dir contains common infra tools such as NGINX ingress controller and Helm repository definitions
- **clusters** dir contains the Flux configuration per cluster

```sh
├── apps
│   ├── base
│   └── overlays
│       ├── staging
│       └── production
├── infrastructure
│   ├── nginx
│   ├── redis
│   └── sources
└── clusters
    ├── staging
    └── production
```

The apps configuration is structured into:

- **apps/base/** dir contains namespaces and Helm release definitions
- **apps/production/** dir contains the production Helm release values
- **apps/staging/** dir contains the staging values

```sh
├── base
│   ├── api
│   │   ├── deployment.yml
│   │   ├── configmap.yml
│   │   ├── hpa.yml
│   │   ├── ingress.yml
│   │   ├── kustomization.yml
│   │   └── service.yml
│   ├── cache
│   │   ├── deployment.yml
│   │   ├── configmap.yml
│   │   ├── hpa.yml
│   │   ├── kustomization.yml
│   │   └── service.yml
│   └── client
│       ├── deployment.yml
│       ├── configmap.yml
│       ├── hpa.yml
│       ├── ingress.yml
│       ├── kustomization.yml
│       └── service.yml
└── overlays
    ├── staging
    │   ├── deployment-patch.yaml
    │   ├── hpa-patch.yaml
    │   ├── kustomization.yaml
    └── production
        ├── deployment-patch.yaml
        ├── hpa-patch.yaml
        └── kustomization.yaml
```

```
./apps/
├── base
│   └── api
│       ├── kustomization.yaml
│       ├── namespace.yaml
│       └── release.yaml
├── production
│   ├── kustomization.yaml
│   └── podinfo-patch.yaml
└── staging
    ├── kustomization.yaml
    └── podinfo-patch.yaml
```

<!-- PREREQUISITES -->
## Prerequisites :hammer_and_wrench:
**NB.** The setup is tested on `macOS Catalina`.

Docker Desktop [installed](https://docs.docker.com/install/)
```sh
# If you don't want to sign up in order to download Docker
# use the following command to download the installer directly
$ curl -s https://download.docker.com/mac/stable/Docker.dmg
```

kubectl (at least version 1.18) [installed](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
```sh
$ brew install kubernetes-cli
```

flux [installed](https://toolkit.fluxcd.io/guides/installation/)
```sh
$ brew install fluxcd/tap/flux
```

kustomize (at least version 3.8.8) [installed](https://github.com/kubernetes-sigs/kustomize/blob/master/docs/INSTALL.md)
```sh
brew install kustomize
```

k3d (at least version v3.4.0) [installed](https://github.com/rancher/k3d)
```sh
$ brew install k3d
```

<!-- USAGE -->
## Usage :keyboard:
Create k3s cluster using k3d:
```sh
$ k3d cluster create gitops-example-staging --agents 2 --update-default-kubeconfig
```
Make sure your KUBECONFIG points to k3s cluster context:
```sh
$ kubectl get nodes
```
Verify that k3s cluster satisfies the prerequisites:
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
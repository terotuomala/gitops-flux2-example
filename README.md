# GitOps workflow example using Flux2
[![Lint](https://github.com/terotuomala/gitops-flux2-example/workflows/Lint/badge.svg)](https://github.com/terotuomala/gitops-flux2-example/actions)

A simple example of managing multiple local K3s clusters including example applications using Flux2, Helm and Kustomize.

<!-- TABLE OF CONTENTS -->
## Table of Contents
* [Features](#rocket-features)
* [Overview](#mag-overview)
* [Example application](#performing_arts-example-application)
* [Flux directory structure](#card_file_box-flux-directory-structure)
  * [Infrastructure](#infrastructure)
  * [Clusters](#clusters)
* [Prerequisites](#hammer_and_wrench-prerequisites)
* [Usage](#keyboard-usage)
* [Thanks](#pray-thanks)

<!-- FEATURES -->
## :rocket: Features
- Two local [K3s](https://github.com/rancher/k3s) clusters using [K3d](https://github.com/rancher/k3d)
- Example applications (separate repositories) including [Single-page Application](https://github.com/terotuomala/k8s-create-react-app-example) and [REST API](https://github.com/terotuomala/k8s-express-api-example)
- Continuous Delivery with GitOps workflow using [Flux2](https://github.com/fluxcd/flux2)
- Scheduled upgrade check of Flux2 using [Renovate](https://docs.renovatebot.com)
- Policies to secure Kubernetes Pods using [Kyverno](https://github.com/kyverno/kyverno)
- Network traffic flow control using [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
- Easy installation using [go-task](https://github.com/go-task/task)
- YAML validation using [yamllint](https://github.com/adrienverge/yamllint)

<!-- OVERVIEW -->
## :mag: Overview
https://github.com/kubernetes/community/tree/master/icons

<!-- EXMAPLE APPLICATION -->
## :performing_arts: Example Application
> The applications are located in separated GitHub repositories and are deployed to the K3s cluster using Flux2.

The example applications  consist from: 
- [Single-page Application](https://github.com/terotuomala/k8s-create-react-app-example) (Create React App)
- [REST API](https://github.com/terotuomala/k8s-express-api-example) with in-memory data store (Node.js + Express.js + Redis)

<!-- FLUX DIRECTORY STRUCTURE -->
## :card_file_box: Flux directory structure
The folders are structured based on the [Flux2 example](https://github.com/fluxcd/flux2-kustomize-helm-example).

```
├── infrastructure
│   ├── calico
│   ├── kyverno
│   ├── kyverno-policies
│   ├── nginx
│   ├── redis
│   └── sources
└── clusters
    ├── cluster-1
    └── cluster-2
```
### Infrastructure
Includes `calico`, `kyverno`, `kyverno-policies`, `nginx` and `redis` configurations as well as `Helm Repository` definitions. It also includes example applications `Git Repository` definitions ([api.yaml](https://github.com/terotuomala/gitops-flux2-example/blob/main/infrastructure/sources/api.yaml) and [client.yaml](https://github.com/terotuomala/gitops-flux2-example/blob/main/infrastructure/sources/client.yaml))

```
└── infrastructure
    ├── calico
    │   └── calico.yaml
    ├── kyverno
    │   └── kustomization.yaml
    ├── kyverno-policies
    │   ├── kustomization.yaml
    │   └── pod-security-restricted.yaml
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
    ├── cluster-1
    │   ├── apps.yaml
    │   └── infrastructure.yaml
    └── cluster-2
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
> **NB.** The setup is tested on `macOS Big Sur`.

The first prerequisite is to install [go-task](https://github.com/go-task/task) in order to make the setup a bit easier:

```sh
brew install go-task/tap/go-task
```

> **NB.** The tasks used in the setup are defined in [Taskfile.yml](https://github.com/terotuomala/gitops-flux2-example/blob/taskfile/Taskfile.yml).

The following prerequisites are used in order to create and manage the local K3s cluster(s):

- [Docker Desktop](https://hub.docker.com/editions/community/docker-ce-desktop-mac/)
- [Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) (at least version 1.18)
- [Flux CLI](https://toolkit.fluxcd.io/guides/installation/)
- [K3d](https://github.com/rancher/k3d) (at least version v4.4.0)

If you don't have them installed yet you can install them using [install-prerequisites](https://github.com/terotuomala/gitops-flux2-example/blob/taskfile/Taskfile.yml#L4) task:

```sh
task install-prerequisites
```

Fork your own copy of this repository to your GitHub account and create a [personal access token](https://docs.github.com/en/github/authenticating-to-github/creating-a-personal-access-token) and export the following variables:
```sh
export GITHUB_TOKEN=<YOUR_PERSONAL_ACCESS_TOKEN>
export GITHUB_USER=<YOUR_GITHUB_USERNAME>
export GITHUB_REPO=<YOUR_FORKED_GITHUB_REPO_NAME>
```

<!-- USAGE -->
## :keyboard: Usage
> **NB.** Both K3s clusters are using Calico instead of Flannel in order to be able to use Network Policies.

### Local K3s cluster-1
Create the cluster:

```sh
task create-cluster1
```

Verify that Calico controller deployment is ready:
```sh
task verify-calico
```

Verify that k3s cluster-1 satisfies flux2 prerequisites:
```sh
task flux-check
```

Install Flux and configure it to manage itself from a Git repository:
```sh
task flux-bootstrap-cluster1
```

Flux2 is configured to deploy content of the `infrastructure` items using Helm before the application. Verify that the infrastructure Helm releases are synchronized to the cluster:
```sh
flux get hr -A
```

Verify that the api and client applications are synchronized to the cluster:
```sh
flux get kustomization -A
```

The example applications should be accessible via Ingress: 
- Single-page Application: `http://localhost:8080`
- REST API: `http://api.localhost:8080`

### Local K3s cluster-2

<details>
  <summary>(Optional)</summary>
  Create the cluster:

  ```sh
  task create-cluster2
  ```

  Verify that Calico controller deployment is ready:
  ```sh
  task verify-calico
  ```

  Verify that k3s cluster-2 satisfies flux2 prerequisites:
  ```sh
  task flux-check
  ```

  Install Flux and configure it to manage itself from a Git repository:
  ```sh
  task flux-bootstrap-cluster2
  ```

  Verify that the infrastructure Helm releases are synchronized to the cluster:
  ```sh
  flux get hr -A
  ```

  Verify that the api and client applications are synchronized to the cluster:
  ```sh
  flux get kustomization -A
  ```

  The example applications should be accessible via Ingress: 
  - Single-page Application: `http://localhost:8081`
  - REST API: `http://api.localhost:8081`
</details>

<!-- THANKS -->
## :pray: Thanks
Inspired by these excellent projects:
- https://github.com/fluxcd/flux2-kustomize-helm-example
- https://github.com/onedr0p/home-cluster

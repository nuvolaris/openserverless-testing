# OpenServerless testing

Since we are testing in many clouds and environments, test setup is pretty complicated. Details are in [this document](SETUP.md), please read it carefully...

## Supported platforms

| Platform | Description | Status |
|---|---|---|
| `kind` | Local Docker-based cluster via `ops setup devcluster` | Active |
| `k3s-amd` | Single AMD VM with k3s installed via `ops setup server` | Active |
| `k3s-arm` | Single ARM VM with k3s installed via `ops setup server` | Active |
| `k8s` | Generic Kubernetes cluster accessed via kubeconfig | Active |
| `mk8s` | MicroK8s on Azure VM | Disabled |
| `eks` | Amazon EKS cluster | Disabled |
| `aks` | Azure AKS cluster | Disabled |
| `gke` | Google GKE cluster | Disabled |
| `osh` | OpenShift on GCP | Disabled |

## Workflows

This repository has three GitHub Actions workflows:

### `operator-pr-test.yaml` — Operator PR Test

Triggered via `repository_dispatch` (event type `operator-pr-test`) from `openserverless-operator`.
Builds an operator Docker image from a PR branch, patches `opsroot.json` to use it, and runs the full acceptance test suite on the specified platform.

The dispatch is initiated from `openserverless-operator` by posting `/testing <platform>` on a PR comment or via manual workflow dispatch. See the [operator README](https://github.com/apache/openserverless-operator#pr-testing-workflow) for details.

### `tests.yaml` — AllTests (tag-based)

Triggered on tag push matching `*-*` (e.g. `k3s-amd-2603041800`). Runs the full acceptance suite sequentially on Linux, Windows, and macOS runners. The platform is extracted from the tag name.

Tags are typically created by the `platform-ci-tests.yaml` workflow below.

### `platform-ci-tests.yaml` — On Tasks Testing Request

Triggered via `repository_dispatch` (event type `olaris-testing-update`). Creates a git tag combining the platform and a timestamp, which in turn triggers the `tests.yaml` workflow above.

## Acceptance Test Status: 103/103

<img src="img/progress.svg" width="60%">

|  |               |Kind|M8S |K3S |EKS |AKS |GKE |OSH |
|--|---------------|----|----|----|----|----|----|----|
|1 |Deploy         | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | 
|2 |SSL            | N/A| ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
|3 |Sys Redis      | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
|4a|Sys FerretDB   | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | 
|4b|Sys Postgres   | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | 
|5 |Sys Minio      | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | 
|6 |Login          | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | 
|7 |Statics        | N/A| ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | 
|8 |User Redis     | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | 
|9a|User FerretDB  | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
|9b|User Postgres  | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | 
|10|User Minio     | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | 
|11|Nuv Win        | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
|12|Nuv Mac        | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
|13|We skip this one | N/A | N/A | N/A | N/A | N/A | N/A | N/A |
|14|Runtimes       | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |



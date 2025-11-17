# 📑 Jenkins Migration: VM to Kubernetes

This document outlines the essential steps and considerations for migrating a Jenkins instance from a Virtual Machine (VM) environment to a Kubernetes cluster, aiming for greater scalability, resilience, and resource utilization.

## 🎯 Overview

The migration involves containerizing the Jenkins controller (master), transferring persistent data (`$JENKINS_HOME`), and configuring ephemeral Pod-based agents.

## 🚀 Phase 1: Preparation

### 1. Containerize the Jenkins Controller
* Create a customized **Docker Image** for the Jenkins Controller.
    * Use the official LTS image as a base.
    * Add necessary tools, custom configurations, and essential plugins (including the **Kubernetes Plugin**).
* **Adopt JCasC:** Implement **Jenkins Configuration as Code** to define the controller's configuration (plugins, security, etc.) via version-controlled YAML files, simplifying the new deployment.

### 2. Kubernetes Cluster Setup
* Create a dedicated **Namespace** for Jenkins resources (e.g., `jenkins`).
* Configure **Persistent Storage**: Define a **PersistentVolume (PV)** and a **PersistentVolumeClaim (PVC)** to mount the `$JENKINS_HOME` directory and ensure persistence for data (jobs, history, plugins, etc.).

### 3. Audit and Cleanup
* Document all configurations, credentials, and plugins from the source Jenkins VM instance.
* Remove unnecessary or old plugins, jobs, and build history to reduce the volume of data that needs to be migrated.

## ⚙️ Phase 2: Deployment on Kubernetes

### 1. Define Manifests
* Create a **Deployment** (or **StatefulSet**) to manage the Jenkins Controller Pod, using your custom Docker image.
* Define a **Service** (e.g., `LoadBalancer` or `NodePort` with an **Ingress**) to expose the Jenkins user interface.
* **Recommended Alternative:** Use the **Jenkins Helm Chart** for a faster and standardized deployment.

### 2. Configure Dynamic Agents
* Install and configure the **Kubernetes Plugin** on the new Jenkins Controller.
* Under **Manage Jenkins** $\rightarrow$ **Nodes and Clouds**, configure a **"Kubernetes Cloud"**.
    * Define **Pod Templates


## Get Repository Info

```console
helm repo add jenkins https://charts.jenkins.io
helm repo update
```

_See [`helm repo`](https://helm.sh/docs/helm/helm_repo/) for command documentation._

## Install Chart

```console
helm install jenkins stable/jenkins -n jenkins
```

Since version `5.6.0` the chart is available as an OCI image and can be installed using:

```console
helm install [RELEASE_NAME] oci://ghcr.io/jenkinsci/helm-charts/jenkins [flags]
```

_See [configuration](#configuration) below._

_See [helm install](https://helm.sh/docs/helm/helm_install/) for command documentation._

## Uninstall Chart

```console
# Helm 3
$ helm uninstall [RELEASE_NAME]
```

This removes all the Kubernetes components associated with the chart and deletes the release.

_See [helm uninstall](https://helm.sh/docs/helm/helm_uninstall/) for command documentation._

## Upgrade Chart

```console
# Helm 3
$ helm upgrade [RELEASE_NAME] jenkins/jenkins [flags]
```

_See [helm upgrade](https://helm.sh/docs/helm/helm_upgrade/) for command documentation._

Visit the chart's [CHANGELOG](https://github.com/jenkinsci/helm-charts/blob/main/charts/jenkins/CHANGELOG.md) to view the chart's release history.
For migration between major version check [migration guide](#migration-guide).

## Building weekly releases

The default charts target Long-Term-Support (LTS) releases of Jenkins.
To use other versions the easiest way is to update the image tag to the version you want.
You can also rebuild the chart if you want the `appVersion` field to match.

## Configuration

See [Customizing the Chart Before Installing](https://helm.sh/docs/intro/using_helm/#customizing-the-chart-before-installing).
To see all configurable options with detailed comments, visit the chart's [values.yaml](https://github.com/jenkinsci/helm-charts/blob/main/charts/jenkins/values.yaml), or run these configuration commands:

```console
# Helm 3
$ helm show values jenkins/jenkins
```

For a summary of all configurable options, see [VALUES.md](https://github.com/jenkinsci/helm-charts/blob/main/charts/jenkins/VALUES.md).



## Migration Guide

### From stable repository

Upgrade an existing release from `stable/jenkins` to `jenkins/jenkins` seamlessly by ensuring you have the latest [repository info](#get-repository-info) and running the [upgrade commands](#upgrade-chart) specifying the `jenkins/jenkins` chart.

### Major Version Upgrades

Chart release versions follow [SemVer](../../CONTRIBUTING.md#versioning), where a MAJOR version change (example `1.0.0` -> `2.0.0`) indicates an incompatible breaking change needing manual actions.

See [UPGRADING.md](./UPGRADING.md) for a list of breaking changes
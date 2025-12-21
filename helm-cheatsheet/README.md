# Devoriales - Helm Cheatsheet

Helm is the package manager for Kubernetes. This guide serves as a comprehensive reference for managing charts, releases, and repositories, from day-to-day operations to advanced chart development.

This cheat sheet is based on the article **The Helm Cheatsheet: Maximizing Kubernetes Deployments**.

**Devoriales Article:** [https://devoriales.com/post/313/the-helm-cheatsheet-maximizing-kubernetes-deployments](https://devoriales.com/post/313/the-helm-cheatsheet-maximizing-kubernetes-deployments)

**Official Documentation:** [helm.sh/docs](https://helm.sh/docs)

## Table of Contents

- [Core Concepts](#core-concepts)
- [Common Workflow](#common-workflow)
- [Repository Management](#repository-management)
- [Searching](#searching)
- [Release Management](#release-management)
- [Information & Debugging](#information--debugging)
- [Chart Development](#chart-development)
- [OCI Registries](#oci-registries)
- [Plugins](#plugins)
- [Helm Challenge](#helm-challenge)

## Core Concepts

- **Chart:** A package of pre-configured Kubernetes resources.
- **Release:** A specific instance of a chart deployed to the cluster.
- **Repository:** A collection of charts.

## Common Workflow (The Happy Path)

Most users will stick to these commands for daily usage.

```bash
# 1. Add a repository
helm repo add bitnami https://charts.bitnami.com/bitnami

# 2. Update your local cache
helm repo update

# 3. Search for a chart
helm search repo wordpress

# 4. Install a chart (creates a release)
helm install my-release bitnami/wordpress

# 5. List running releases
helm list

# 6. Uninstall a release
helm uninstall my-release
```

## Repository Management

Manage the locations where you download charts from.

| Command | Description |
|---------|-------------|
| `helm repo add <name> <url>` | Add a chart repository. |
| `helm repo list` | List added chart repositories. |
| `helm repo update` | Update local cache of available charts from all added repos. |
| `helm repo remove <name>` | Remove a repository. |
| `helm repo index <dir>` | Generate an index.yaml file for a directory containing packaged charts. |

## Searching

Find charts in your configured repositories or the artifact hub.

| Command | Description |
|---------|-------------|
| `helm search repo <keyword>` | Search for charts in your added repositories (displays latest stable version). |
| `helm search hub <keyword>` | Search for charts on Artifact Hub (online). |
| `helm search repo <keyword> --versions` | List all available versions of a specific chart. |
| `helm search repo <keyword> --devel` | Search including development/latest versions (alpha, beta, rc). |

## Release Management

Manage the lifecycle of applications deployed to your cluster.

### Install & Upgrade

| Command | Description |
|---------|-------------|
| `helm install <name> <chart>` | Install a chart. |
| `helm install <name> <chart> --namespace <ns> --create-namespace` | Install into a specific namespace, creating it if needed. |
| `helm install <name> <chart> --values values.yaml` | Install using a custom values file. |
| `helm install <name> <chart> --set key=value` | Install and override specific values on the CLI. |
| `helm upgrade <release> <chart>` | Upgrade an existing release. |
| `helm upgrade <release> <chart> --install` | **Pro Tip:** Upgrades a release, or installs it if it doesn't exist (idempotent). |
| `helm upgrade <release> <chart> --atomic --cleanup-on-fail` | Upgrade and rollback automatically if it fails. |

### List & Delete

| Command | Description |
|---------|-------------|
| `helm list` | List releases in the current namespace. |
| `helm list -A` | List releases across all namespaces. |
| `helm list --filter 'arelease.*'` | List releases matching a regex filter. |
| `helm uninstall <release>` | Remove a release (deletes all k8s resources). |
| `helm uninstall <release> --keep-history` | Remove a release but keep the history (allows viewing old logs/rollback). |

### Rollback & History

| Command | Description |
|---------|-------------|
| `helm history <release>` | Show revision history for a release. |
| `helm rollback <release> <revision>` | Roll back a release to a specific revision number. |
| `helm status <release>` | Show the status of a specific release. |

## Information & Debugging

Inspect charts and troubleshoot deployments.

| Command | Description |
|---------|-------------|
| `helm get all <release>` | Print all information (manifests, values, notes) for a release. |
| `helm get manifest <release>` | Print the actual Kubernetes YAML running in the cluster. |
| `helm get values <release>` | Print the user-supplied values for a release. |
| `helm show chart <chart>` | Show the Chart.yaml content (metadata). |
| `helm show values <chart>` | Show the default values.yaml for a chart. |
| `helm template <name> <chart>` | Render the chart templates locally without installing (good for debugging). |
| `helm install ... --dry-run --debug` | Simulate an install and print the generated manifests. |

## Chart Development

Commands for creating and working on your own charts.

| Command | Description |
|---------|-------------|
| `helm create <name>` | Create a new chart directory with the recommended structure. |
| `helm lint <path>` | Check a chart for potential issues. |
| `helm package <path>` | Package a chart directory into a .tgz archive. |
| `helm dependency update <path>` | Update dependencies based on Chart.yaml. |

### Creating & Templating Example

#### 1. Scaffolding

Run `helm create mychart` to generate the standard structure:

- **Chart.yaml:** Metadata (name, version).
- **values.yaml:** Default configuration values.
- **templates/:** Directory for K8s YAML files combined with Go template logic.

#### 2. Using Values (values.yaml)

**values.yaml:**

```yaml
replicaCount: 3
image:
  repository: nginx
```

**templates/deployment.yaml:**

```yaml
replicas: {{ .Values.replicaCount }}
image: "{{ .Values.image.repository }}"
```

#### 3. Using Helpers (_helpers.tpl)

**templates/_helpers.tpl:**

```go
{{- define "mychart.fullname" -}}
{{- printf "%s-%s" .Release.Name .Chart.Name | trunc 63 | trimSuffix "-" -}}
{{- end -}}
```

**templates/service.yaml:**

```yaml
metadata:
  name: {{ include "mychart.fullname" . }}
```

## OCI Registries (Helm 3.8+)

Using container registries (like Docker Hub, ECR, GCR) to store charts.

| Command | Description |
|---------|-------------|
| `helm registry login <host>` | Log in to a registry. |
| `helm push <chart>.tgz oci://<host>/<org>` | Push a packaged chart to an OCI registry. |
| `helm pull oci://<host>/<org>/<chart>` | Pull a chart from an OCI registry. |
| `helm pull oci://.../chart --untar` | Pull and unpack the chart content to a local directory. |

## Plugins

Plugins are tools that "plug in" to Helm to add new features.

### Real World Example: helm-diff

See exactly what will change in the YAML before you upgrade.

```bash
# 1. Install the plugin
helm plugin install https://github.com/databus23/helm-diff

# 2. See what would change if you upgraded
helm diff upgrade my-release bitnami/wordpress
```

## Helm Challenge

**Are you ready to steer the ship?**

Kubernetes is complex, but Helm makes it manageable... if you know how to use it! We have prepared a challenge to test if you are a true Helm skipper or still a deckhand.

### What You'll Prove

- **Deployment Strategies:** Handle complex installs, overrides, and upgrades.
- **Deep Diagnostics:** Inspect and troubleshoot charts before deployment.
- **Lifecycle Mastery:** Control the full history and versioning of your applications.
- **Chart Architecture:** Understand how charts are built, configured, and templated.

**Take the Quiz Here**
[Helm Cheatsheet Challenge](https://devoriales.com/quiz/16)  

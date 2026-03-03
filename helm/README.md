# ⛵ Helm

> Frequently used Helm commands for managing Kubernetes packages.

[← Back to Home](../README.md)

---

## 📋 Table of Contents

- [Installation](#installation)
- [Repositories](#repositories)
- [Charts](#charts)
- [Releases](#releases)
- [Upgrades & Rollbacks](#upgrades--rollbacks)
- [Templating & Debugging](#templating--debugging)
- [Chart Development](#chart-development)
- [Plugins](#plugins)

---

## Installation

```bash
# Install Helm (Linux)
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Verify
helm version

# Completion
helm completion bash | sudo tee /etc/bash_completion.d/helm
```

---

## Repositories

```bash
# Add a repo
helm repo add stable https://charts.helm.sh/stable
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx

# List repos
helm repo list

# Update repos
helm repo update

# Remove repo
helm repo remove stable

# Search charts in repos
helm search repo nginx
helm search repo bitnami/nginx --versions
```

---

## Charts

```bash
# Search Artifact Hub
helm search hub wordpress

# Show chart info
helm show all bitnami/nginx
helm show chart bitnami/nginx
helm show values bitnami/nginx

# Download chart
helm pull bitnami/nginx
helm pull bitnami/nginx --untar          # extract
helm pull bitnami/nginx --version 15.0.0
```

---

## Releases

```bash
# Install a chart
helm install myrelease bitnami/nginx
helm install myrelease bitnami/nginx -n myapp
helm install myrelease bitnami/nginx --create-namespace -n myapp
helm install myrelease ./mychart           # local chart
helm install myrelease bitnami/nginx --set service.type=NodePort
helm install myrelease bitnami/nginx -f values-prod.yaml

# List releases
helm list
helm list -n myapp
helm list -A                              # all namespaces

# Get release status
helm status myrelease -n myapp

# Get release values
helm get values myrelease -n myapp
helm get values myrelease -n myapp --all

# Get release manifest
helm get manifest myrelease -n myapp

# Get release notes
helm get notes myrelease

# Uninstall a release
helm uninstall myrelease -n myapp
helm uninstall myrelease --keep-history   # keep history for rollback
```

---

## Upgrades & Rollbacks

```bash
# Upgrade a release
helm upgrade myrelease bitnami/nginx -n myapp
helm upgrade myrelease bitnami/nginx -f values.yaml -n myapp
helm upgrade myrelease bitnami/nginx --set replicaCount=3 -n myapp
helm upgrade myrelease bitnami/nginx --version 15.0.0 -n myapp

# Install or upgrade (idempotent)
helm upgrade --install myrelease bitnami/nginx -n myapp --create-namespace

# Release history
helm history myrelease -n myapp

# Rollback to previous release
helm rollback myrelease -n myapp
helm rollback myrelease 2 -n myapp        # rollback to revision 2
```

---

## Templating & Debugging

```bash
# Render templates without installing
helm template myrelease bitnami/nginx
helm template myrelease ./mychart -f values.yaml

# Dry-run (server-side validation)
helm install myrelease bitnami/nginx --dry-run

# Lint a chart
helm lint ./mychart

# Debug rendering
helm install myrelease ./mychart --debug --dry-run
```

---

## Chart Development

```bash
# Create a new chart
helm create mychart

# Chart structure:
# mychart/
# ├── Chart.yaml          # chart metadata
# ├── values.yaml         # default values
# ├── charts/             # dependencies
# ├── templates/          # Kubernetes manifests
# │   ├── deployment.yaml
# │   ├── service.yaml
# │   ├── ingress.yaml
# │   ├── _helpers.tpl    # named templates
# │   └── NOTES.txt       # post-install notes
# └── .helmignore

# Package a chart
helm package ./mychart
helm package ./mychart --version 1.2.0

# Publish to OCI registry
helm push mychart-1.0.0.tgz oci://registry.example.com/charts

# Pull from OCI registry
helm pull oci://registry.example.com/charts/mychart --version 1.0.0

# Add dependencies (Chart.yaml)
# dependencies:
#   - name: postgresql
#     version: "12.x.x"
#     repository: "https://charts.bitnami.com/bitnami"

# Update dependencies
helm dependency update ./mychart

# Build dependencies
helm dependency build ./mychart
```

**`Chart.yaml` example:**

```yaml
apiVersion: v2
name: mychart
description: My application Helm chart
type: application
version: 0.1.0
appVersion: "1.0.0"
dependencies:
  - name: postgresql
    version: "12.x.x"
    repository: "https://charts.bitnami.com/bitnami"
```

---

## Plugins

```bash
# List installed plugins
helm plugin list

# Install a plugin
helm plugin install https://github.com/databus23/helm-diff

# Update a plugin
helm plugin update diff

# Remove a plugin
helm plugin remove diff

# Popular plugins:
# helm-diff:   show diff between release and upgrade
# helm-secrets: manage encrypted secrets with SOPS
# helm-unittest: unit tests for charts
```

---

[← Back to Home](../README.md)

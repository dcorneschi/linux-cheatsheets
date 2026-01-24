# Helm Cheatsheet

## Overview
Helm is the package manager for Kubernetes that helps you define, install, and upgrade complex Kubernetes applications using charts.

## Directory Structure

```
my-chart/
├── Chart.yaml              # Chart metadata (name, version, description)
├── values.yaml             # Default configuration values
├── values.schema.json      # JSON schema that validates values
├── templates/              # Kubernetes manifest templates
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── configmap.yaml
│   └── _helpers.tpl        # Template helpers (prefixed with _)
├── charts/                 # Dependency charts (optional)
└── README.md               # Documentation
```

## Actual Charts

When you run `helm repo update`, Helm downloads **only the index** (metadata about available charts), not the actual chart packages. Charts are only downloaded when you explicitly:

- `helm pull` - Download chart to local directory
- `helm install` - Download and install (cached temporarily)
- `helm upgrade` - Download new version (cached temporarily)

## Install Helm

```bash
# Install via script (Linux/macOS)
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Install via package managers (macOS)
brew install helm

# Verify installation
helm version
```

## Repository Management

```bash
# Add a repository
helm repo add metrics-server https://kubernetes-sigs.github.io/metrics-server/

# Update repository indexes
helm repo update

# Force update all repositories
helm repo update --fail-on-repo-update-fail

# List configured repositories
helm repo list
helm repo list -o yaml

# Remove a repository
helm repo remove <repo-name>

# Search for charts in repositories
helm search repo traefik

# List all versions with default column width
helm search repo traefik/traefik --versions

# List all versions with full width output (recommended)
helm search repo traefik/traefik --versions --max-col-width 0

# Search Helm Hub
helm search hub metrics-server

# Show chart metadata
helm show chart traefik/traefik

# Show default values
helm show values traefik/traefik

# Show README
helm show readme traefik/traefik

# Show all info
helm show all traefik/traefik
```

## Install Charts

```bash
# Basic installation (default namespace)
helm install metrics-server metrics-server/metrics-server

# Install in specific namespace
helm install metrics-server metrics-server/metrics-server --namespace kube-system

# Install and create the namespace
helm install traefik traefik/traefik -n traefik --create-namespace

# Install with inline values
helm install traefik traefik/traefik -n traefik --create-namespace --set service.type=LoadBalancer

# Install with values file
helm install traefik traefik/traefik -f values.yaml

# Install from local chart
helm install traefik ./traefik

# Install from packaged chart
helm install traefik ./traefik-37.3.0.tgz

# Install with specific version
helm install metrics-server metrics-server/metrics-server --version 3.12.2

# Wait for deployment to complete
helm install metrics-server metrics-server/metrics-server --wait --timeout 5m

# Dry run & debug installation
helm install traefik traefik/traefik --dry-run
helm install traefik traefik/traefik --dry-run --debug
```

## Upgrade Releases

```bash
# Upgrade release
helm upgrade metrics-server metrics-server/metrics-server -n kube-system

## Upgrade to Specific Version
helm upgrade metrics-server metrics-server/metrics-server -n kube-system --version 3.12.2

# Upgrade with inline values
helm upgrade metrics-server metrics-server/metrics-server -n kube-system --set replicas=2

# Upgrade release with new values
helm upgrade metrics-server metrics-server/metrics-server -n kube-system --values values.yaml

# Upgrade and rollback on failure
helm upgrade metrics-server metrics-server/metrics-server -n kube-system --atomic

# Upgrade and wait for completion
helm upgrade metrics-server metrics-server/metrics-server -n kube-system --wait --timeout 5m

# Dry run upgrade
helm upgrade metrics-server metrics-server/metrics-server -n kube-system --dry-run

# Force update (recreate pods)
helm upgrade metrics-server metrics-server/metrics-server -n kube-system --force

# Upgrade and install if not exists
helm upgrade --install metrics-server metrics-server/metrics-server -n kube-system
```

## Rollback

```bash
# Check upgrade history
helm history metrics-server -n kube-system

# Rollback to previous release
helm rollback metrics-server -n kube-system

# Rollback to specific revision
helm rollback metrics-server -n kube-system 1

# Rollback with cleanup hooks
helm rollback metrics-server -n kube-system 1 --cleanup-on-fail
```

## Uninstall

```bash
# Uninstall release
helm uninstall metrics-server

# Uninstall from specific namespace
helm uninstall metrics-server -n kube-system

# Keep release history (soft delete)
helm uninstall metrics-server -n kube-system --keep-history
```

## Download to Current Directory

```bash
# Download chart package (tgz) to current directory
helm pull traefik/traefik

# Download and extract
helm pull traefik/traefik --untar

# Download to specific location
mkdir -p ~/helm-charts
helm pull traefik/traefik --destination ~/helm-charts

# Download and extract to specific location
helm pull traefik/traefik --destination ~/helm-charts --untar

# Download specific version
helm pull traefik/traefik --version 27.0.0 --destination ~/helm-charts

# Download Multiple Charts
mkdir -p ~/helm-charts/{traefik,nginx,postgres}
helm pull traefik/traefik --destination ~/helm-charts/traefik --untar
helm pull bitnami/nginx --destination ~/helm-charts/nginx --untar
helm pull bitnami/postgresql --destination ~/helm-charts/postgres --untar
```

## Create & Develop Charts

```bash
# Create a new chart
helm create my-chart

# Package chart into tarball
helm package my-chart

# Validate chart syntax
helm lint my-chart

# Validate chart values
helm lint my-chart --values values.yaml

# Template rendering (dry run)
helm template metrics-server metrics-server/metrics-server
helm template metrics-server metrics-server/metrics-server --set replicas=2
helm template metrics-server metrics-server/metrics-server --values values.yaml

# Template with debug values
helm template metrics-server metrics-server/metrics-server --debug

# Template to file
helm template metrics-server metrics-server/metrics-server > metrics-server-rendered.yaml
```

## Release Management

Release information is stored in Kubernetes, not on your local machine.

```bash
# List releases in specific namespace
helm list -n argo

# List all releases across all namespaces
helm list -A                     # default --output table
helm list -A --output yaml
helm list -A --output json

# Get specific field
helm list -A --output json | jq '.[] | .name'

# Release data is stored in Kubernetes secrets
kubectl get secrets -n <namespace> -l owner=helm

# Show release details
helm status argo -n argo

# Get user-supplied values
helm get values argo -n argo

# Get all of the values used by a release (including the defaults)
helm get values argo -n argo --all

# Get release manifest (rendered YAML)
helm get manifest argo -n argo

# Get manifest at specific revision
helm get manifest metrics-server -n kube-system --revision 2

# Get release notes
helm get notes argo -n argo

# Get all the information about a named release
helm get all argo -n argo

# Show release history
helm history argo -n argo
helm history argo -n argo --output json

# Compare releases
helm diff revision metrics-server -n kube-system 1 2        # (requires helm-diff plugin)
```

## View Helm Environment Variables

```bash
# Show all Helm environment variables
helm env

# Check cache location
helm env | grep CACHE

# Common Helm paths
helm env | grep -E "(CACHE|CONFIG|DATA)"
```

##  Set Custom Cache Location

```bash
# Set custom cache directory
export HELM_CACHE_HOME=~/my-custom-helm-cache

# Set custom config directory
export HELM_CONFIG_HOME=~/.config/helm

# Set custom data directory
export HELM_DATA_HOME=~/.local/share/helm

# Make permanent (add to ~/.bash_profile or ~/.zshrc)
echo 'export HELM_CACHE_HOME=~/my-custom-helm-cache' >> ~/.bash_profile
```

## List Cached Repository Indexes (macOS)

```bash
# Helm configuration directory
~/Library/Preferences/helm/

# List all files in repository cache
ls -la ~/Library/Caches/helm/repository/

# View with human-readable sizes
ls -lh ~/Library/Caches/helm/repository/

# Show only YAML index files
ls -la ~/Library/Caches/helm/repository/*.yaml

# Show only chart archives
ls -la ~/Library/Caches/helm/repository/*.tgz

# Repository configuration
~/Library/Preferences/helm/repositories.yaml

# Plugin directory
~/Library/helm/plugins/

# Check cache directory size
du -sh ~/Library/Caches/helm/

# Check repository cache size
du -sh ~/Library/Caches/helm/repository/

# List files by size
du -h ~/Library/Caches/helm/repository/* | sort -h

# Remove all cached charts (keeps repository indexes)
rm -f ~/Library/Caches/helm/repository/*.tgz

# Remove specific cached chart
rm -f ~/Library/Caches/helm/repository/traefik-*.tgz

# Clear entire cache (nuclear option)
rm -rf ~/Library/Caches/helm/

# After clearing, update repositories
helm repo update
```

## Examples

### Extract and Modify Chart

```bash
# Download and extract chart
helm pull traefik/traefik --untar --destination ~/my-custom-charts

# Navigate to chart
cd ~/my-custom-charts/traefik

# Modify values or templates
vim values.yaml
vim templates/deployment.yaml

# Create custom chart package
cd ..
helm package traefik
# Creates: traefik-27.0.0.tgz

# Install custom chart
helm install my-traefik ./traefik-27.0.0.tgz
```
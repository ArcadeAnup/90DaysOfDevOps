# Day 51: Helm - Package Manager for Kubernetes

## The Problem Helm Solves

### Without Helm (Bad)
Need to deploy Jenkins to Kubernetes

↓

Write Deployment YAML (Pods, replicas)

Write Service YAML (expose it)

Write ConfigMap YAML (configuration)

Write Secret YAML (credentials)

Write PersistentVolume YAML (storage)

Write Ingress YAML (external access)

... 15+ more files ...

↓

Apply all files in correct order

↓

Hope nothing breaks

↓

To update: manually edit files, reapply

Tedious. Error-prone. Not reusable.

### With Helm (Good)
helm install jenkins jenkins/jenkins

↓

One command deploys entire application

↓

All YAML files already written

↓

Everything pre-configured

↓

To update: helm upgrade jenkins jenkins/jenkins

Simple. Reliable. Reusable.

## What is Helm?

Helm = Package manager for Kubernetes

**npm** = JavaScript packages
**apt** = Linux packages
**Helm** = Kubernetes packages (Charts)

Helm packages Kubernetes manifests into reusable bundles.

## Helm Concepts

### Chart
Pre-written Kubernetes manifests bundled together.
Example: `jenkins/jenkins` chart contains all Jenkins YAML files.

### Repository
Collection of Charts (like package registry).
Example: `https://charts.jenkins.io` hosts Jenkins charts.

### Release
Deployed instance of a Chart.
Example: `jenkins-release` = one running instance of Jenkins chart.

### Values
Customization parameters for Charts.
Example: `replicas: 3`, `persistence: enabled`, `adminPassword: secret`

## Helm Installation

### Step 1: Install Helm CLI

```bash
# macOS
brew install helm

# Linux
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Verify
helm version
# version.BuildInfo{Version:"v3.12.0", ...}
```

### Step 2: Add Repository

Repositories host Charts (like GitHub for Helm).

```bash
# Add Jenkins repository
helm repo add jenkins https://charts.jenkins.io

# List repositories
helm repo list
# NAME     URL
# jenkins  https://charts.jenkins.io

# Update repositories (fetch latest charts)
helm repo update
```

## Deploying Jenkins with Helm

### One-Line Deploy

```bash
helm install jenkins jenkins/jenkins
```

**What happens:**
1. Helm downloads Jenkins chart from repository
2. Reads default values
3. Generates all YAML files
4. Applies them to cluster
5. Creates Jenkins deployment

Verify:

```bash
# List releases
helm list
# NAME     NAMESPACE  REVISION  STATUS      CHART           APP VERSION  AGE
# jenkins  default    1         deployed    jenkins-3.11.0  2.375        1m

# Check Pods
kubectl get pods
# NAME              READY   STATUS    RESTARTS
# jenkins-0         1/1     Running   0

# Check Services
kubectl get svc
# jenkins  ClusterIP  10.96.0.5  <none>  8080/TCP,50000/TCP
```

Jenkins running with one command.

## Customizing with Values

Charts have configurable values. Instead of editing YAML, pass values to Helm.

### View Default Values

```bash
helm show values jenkins/jenkins | head -50
```

Output shows all configurable options (replicas, persistence, admin password, etc.).

### Deploy with Custom Values

#### Option 1: Command Line

```bash
helm install jenkins jenkins/jenkins \
  --set controller.adminPassword=mypassword \
  --set controller.replicas=2 \
  --set persistence.enabled=true \
  --set persistence.size=10Gi
```

#### Option 2: Values File

Create `values.yaml`:

```yaml
controller:
  adminPassword: mypassword
  replicas: 2

persistence:
  enabled: true
  size: 10Gi

serviceType: LoadBalancer
```

Deploy:

```bash
helm install jenkins jenkins/jenkins -f values.yaml
```

Much cleaner than command line.

## Common Helm Commands

### Install

```bash
# Simple install
helm install jenkins jenkins/jenkins

# Install with custom values
helm install jenkins jenkins/jenkins -f values.yaml

# Install with namespace
helm install jenkins jenkins/jenkins -n ci-cd
```

### Upgrade

```bash
# Upgrade to newer chart version
helm upgrade jenkins jenkins/jenkins

# Upgrade with new values
helm upgrade jenkins jenkins/jenkins -f values.yaml
```

### Rollback

```bash
# Rollback to previous version
helm rollback jenkins 1

# List revisions
helm history jenkins
# REVISION  STATUS       CHART            APP VERSION  DESCRIPTION
# 1         superseded   jenkins-3.11.0   2.375        Install complete
# 2         deployed     jenkins-3.12.0   2.376        Upgrade complete
```

### Uninstall

```bash
# Remove release (deletes all resources)
helm uninstall jenkins

# Keep ConfigMaps/Secrets for backup
helm uninstall jenkins --keep-history
```

## Viewing Installed Release

```bash
# See release details
helm show chart jenkins/jenkins

# See what YAML will be generated
helm template jenkins jenkins/jenkins

# See actual deployment
kubectl get all -l app.kubernetes.io/instance=jenkins
```



## Creating Your Own Chart (High Level)

Chart structure:
my-chart/

├── Chart.yaml           # Chart metadata

├── values.yaml          # Default values

├── templates/

│   ├── deployment.yaml

│   ├── service.yaml

│   ├── configmap.yaml

│   └── ingress.yaml

└── README.md

### Chart.yaml

```yaml
apiVersion: v2
name: my-app
description: My Application
type: application
version: 1.0.0
appVersion: "1.0"
```

### values.yaml

```yaml
replicaCount: 1

image:
  repository: my-app
  tag: "1.0"

service:
  type: ClusterIP
  port: 80

persistence:
  enabled: false
```

### templates/deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Chart.Name }}
spec:
  replicas: {{ .Values.replicaCount }}
  template:
    spec:
      containers:
      - name: app
        image: {{ .Values.image.repository }}:{{ .Values.image.tag }}
```

Deploy:

```bash
helm install my-app ./my-chart -f values.yaml
```

Helm substitutes `{{ .Values.key }}` with actual values.

## Popular Helm Charts
bitnami/  - Database, caching, infrastructure

prometheus/ - Monitoring

ingress-nginx/ - NGINX Ingress Controller

elasticsearch/ - Search engine

redis/ - Cache

postgresql/ - Database

mysql/ - Database

grafana/ - Dashboards

For any popular application, search Helm Hub: `https://artifacthub.io`

## Why Helm Matters

1. **Reusability** - One chart, deploy anywhere
2. **Consistency** - Same deployment every time
3. **Simplicity** - One command instead of 20 files
4. **Community** - Thousands of pre-built charts
5. **Upgrades** - Easy to update with `helm upgrade`
6. **Rollback** - Quick rollback if things break


## High-Level Understanding

**What:** Package manager for Kubernetes
**Why:** Deploy complex apps with one command
**How:** Charts contain pre-written YAML, values customize them
**When:** Deploying known applications (databases, monitoring, CI/CD)
**Example:** `helm install jenkins jenkins/jenkins` deploys entire Jenkins

---

**Progress: 51/90 days complete**



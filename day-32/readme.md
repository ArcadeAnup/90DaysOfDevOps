# Day 32: Kubernetes Deployments and Managed Services

## Pods vs Deployments

### Pods (Day 31)
- Single container wrapper
- Ephemeral (if it dies, it's gone)
- No self-healing
- No scaling

### Deployments (Day 32)
- Manages multiple Pod replicas
- Self-healing (replaces failed Pods)
- Auto-scaling support
- Rolling updates (zero-downtime deployments)
- Actual production unit

**TL;DR:** Pods are for learning. Deployments are for real applications.

## Creating a Deployment

### YAML Manifest

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 4                    # Want 4 running at all times
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

### Deploy It

```bash
kubectl apply -f deployment.yaml
```

### Verify

```bash
# See Deployment
kubectl get deployments

# See Pods (should be 4)
kubectl get pods

# See details
kubectl describe deployment nginx-deployment

# Watch real-time
kubectl get pods -w
```

**Output:**
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-7d9f8f7f9f-abc12   1/1     Running   0          2m
nginx-deployment-7d9f8f7f9f-def45   1/1     Running   0          2m
nginx-deployment-7d9f8f7f9f-ghi67   1/1     Running   0          2m
nginx-deployment-7d9f8f7f9f-jkl89   1/1     Running   0          2m

4 Pods running.

## Self-Healing in Action

### Delete a Pod

```bash
kubectl delete pod nginx-deployment-7d9f8f7f9f-abc12
```

### Watch Immediately

```bash
kubectl get pods -w

# You'll see:
# Old Pod: Terminating
# New Pod: Pending → Running
```

Kubernetes automatically creates a replacement. Always maintains 4 replicas.

## Scaling Deployments

### Scale Up

```bash
kubectl scale deployment nginx-deployment --replicas=10
```

10 Pods now running (6 new ones created).

### Scale Down

```bash
kubectl scale deployment nginx-deployment --replicas=2
```

Back to 2 Pods (4 pods terminated).

### Or Update YAML

```yaml
spec:
  replicas: 6    # Change this
```

Then: `kubectl apply -f deployment.yaml`

## Managed Kubernetes Services

You can run Kubernetes yourself (hard) or let cloud providers manage it.

### EKS (Amazon Elastic Kubernetes Service)
- AWS manages control plane
- You manage worker nodes
- Integration with AWS services (RDS, S3, etc.)
- Pay for nodes + management fee


### AKS (Azure Kubernetes Service)
- Microsoft manages control plane
- You manage nodes
- Integrates with Azure services

### Self-Hosted Kubernetes
- You manage everything (control plane + nodes)
- Maximum control
- Operational burden
- Use case: large companies with DevOps teams



## Deployment Lifecycle

Create Deployment YAML
↓
kubectl apply -f deployment.yaml
↓
Kubernetes creates ReplicaSet (manages Pods)
↓
ReplicaSet creates N Pods
↓
Pods run your containers
↓
If Pod fails → ReplicaSet creates new one
↓
If you update image → Rolling update (old Pods replaced gradually)


## Rolling Updates (Zero Downtime)

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1          # 1 extra Pod during update
      maxUnavailable: 0    # 0 Pods down during update
```

When you update image:
- Create 1 new Pod with new image
- Old Pod serving traffic still
- When new Pod healthy, terminate old one
- Repeat for all replicas
- Zero downtime

## Common Deployment Commands

```bash
# Create Deployment
kubectl apply -f deployment.yaml

# Get Deployments
kubectl get deployments

# Get Pods for Deployment
kubectl get pods -l app=nginx

# Describe Deployment
kubectl describe deployment nginx-deployment

# Update image
kubectl set image deployment/nginx-deployment \
  nginx=nginx:1.20

# Rollback to previous version
kubectl rollout undo deployment/nginx-deployment

# Check rollout status
kubectl rollout status deployment/nginx-deployment

# Scale
kubectl scale deployment nginx-deployment --replicas=5

# Delete Deployment
kubectl delete deployment nginx-deployment
```

**Progress: 32/90 days complete**

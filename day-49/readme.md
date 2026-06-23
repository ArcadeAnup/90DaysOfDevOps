# Day 49: Horizontal Pod Autoscaler (HPA)

## The Problem HPA Solves

### Manual Scaling (Bad)
Traffic increases

↓

System slows down

↓

Engineer notices (maybe 10 minutes later)

↓

Manually scales Pods: kubectl scale deployment --replicas=10

↓

Pods start up

↓

Traffic now handled

Reactive. Slow. Misses traffic spikes.

### Automatic Scaling (Good)
Traffic increases

↓

CPU usage spikes

↓

HPA detects (in seconds)

↓

Automatically scales Pods

↓

Pods start up

↓

Traffic handled

Proactive. Fast. Handles spikes automatically.

## What is HPA?

HPA = Horizontal Pod Autoscaler

Watches metrics (CPU, memory, custom), automatically scales number of Pods up/down to meet demand.

**Horizontal** = add more Pods (not make existing ones bigger)
**Autoscaler** = automatically scales based on metrics

## How It Works
HPA Controller (watches metrics)

↓

Checks Pod resource usage (CPU, memory)

↓

Compares to defined thresholds

↓

Scales Deployment replicas up/down

↓

Desired state reached

Continuous loop. Every 15 seconds (default) it checks metrics.

## Creating an HPA

### Prerequisites

Pods must have resource requests defined:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web
  template:
    spec:
      containers:
      - name: app
        image: nginx:latest
        resources:
          requests:
            cpu: 100m          # Request 100 millicores
            memory: 128Mi      # Request 128 megabytes
          limits:
            cpu: 500m          # Max 500 millicores
            memory: 512Mi      # Max 512 megabytes
```

HPA uses these requests to calculate CPU percentage.

### Create HPA

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-app              # Target this Deployment
  minReplicas: 2               # Minimum Pods
  maxReplicas: 10              # Maximum Pods
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70 # Scale up when CPU > 70%
```

**What this means:**
- Watch `web-app` Deployment
- Keep 2-10 Pods running
- If CPU average > 70%, add Pods
- If CPU average < 70%, remove Pods

### Apply HPA

```bash
kubectl apply -f hpa.yaml

# Verify
kubectl get hpa
# NAME          REFERENCE           TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
# web-app-hpa   Deployment/web-app  45%/70%   2         10        2          1m
```

Output shows:
- Current CPU: 45%
- Target CPU: 70%
- Current replicas: 2
- Min/Max: 2-10

## How HPA Decides to Scale

### Scale Up Decision
Current replicas: 2

Current CPU: 85%

Target CPU: 70%
Calculation:

Desired replicas = current × (current CPU / target CPU)

Desired replicas = 2 × (85 / 70)

Desired replicas = 2.43 → rounds to 3
Action: Scale to 3 replicas

### Scale Down Decision
Current replicas: 5

Current CPU: 30%

Target CPU: 70%
Calculation:

Desired replicas = 5 × (30 / 70)

Desired replicas = 2.14 → rounds to 2
Action: Scale down to 2 replicas (minimum is 2, so stays at 2)

### Cooldown Period

After scaling:
- Scale up cooldown: 3 minutes (wait before scaling up again)
- Scale down cooldown: 5 minutes (wait before scaling down again)

Prevents flapping (rapid up/down).

## Metrics HPA Can Use

### CPU (Most Common)

```yaml
metrics:
- type: Resource
  resource:
    name: cpu
    target:
      type: Utilization
      averageUtilization: 70
```

Scales based on CPU usage.

### Memory

```yaml
metrics:
- type: Resource
  resource:
    name: memory
    target:
      type: Utilization
      averageUtilization: 80
```

Scales based on memory usage.

### Custom Metrics

```yaml
metrics:
- type: Pods
  pods:
    metric:
      name: http_requests_per_second
    target:
      type: AverageValue
      averageValue: 1000
```

Requires custom metrics provider (Prometheus, etc.).

### Multiple Metrics

```yaml
metrics:
- type: Resource
  resource:
    name: cpu
    target:
      type: Utilization
      averageUtilization: 70
- type: Resource
  resource:
    name: memory
    target:
      type: Utilization
      averageUtilization: 80
```

Scales when **any** metric exceeds threshold.

## HPA Versions

### v1 (Simple, older)
```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: hpa
spec:
  scaleTargetRef:
    kind: Deployment
    name: app
  minReplicas: 1
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70
```

Only CPU metric.

### v2 (Advanced, newer)
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: hpa
spec:
  scaleTargetRef:
    kind: Deployment
    name: app
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

Multiple metrics, more control.

## Monitoring HPA Activity

```bash
# Check HPA status
kubectl get hpa

# Detailed info
kubectl describe hpa web-app-hpa
# Shows: current metrics, replicas, events, scaling decisions

# Watch real-time
kubectl get hpa -w
# Shows metrics changing in real-time
```

## Real-World Example: Web Application

```yaml
---
# Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-server
spec:
  replicas: 2
  selector:
    matchLabels:
      app: api
  template:
    metadata:
      labels:
        app: api
    spec:
      containers:
      - name: api
        image: my-api:latest
        resources:
          requests:
            cpu: 200m
            memory: 256Mi
          limits:
            cpu: 500m
            memory: 512Mi
        ports:
        - containerPort: 8000

---
# Service
apiVersion: v1
kind: Service
metadata:
  name: api-service
spec:
  selector:
    app: api
  ports:
  - port: 80
    targetPort: 8000
  type: LoadBalancer

---
# HPA
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api-server
  minReplicas: 2
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 75
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
      - type: Percent
        value: 100
        periodSeconds: 30
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 10
        periodSeconds: 60
```

Deploy:
```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl apply -f hpa.yaml

# Watch HPA in action
kubectl get hpa -w
```

As traffic increases:
- CPU/memory rises above 75%/80%
- HPA detects it
- Scales Pods up (2 → 3 → 5 → 10, etc.)
- New Pods handle traffic
- CPU/memory back to normal
- HPA settles

Traffic decreases:
- CPU/memory drops below 75%/80%
- HPA detects it
- Waits 5 minutes (cooldown)
- Scales Pods down (10 → 5 → 3 → 2)
- Cost optimized

## Requirements for HPA

1. **Metrics Server** must be installed (collects Pod metrics)
```bash
kubectl get deployment metrics-server -n kube-system
```

2. **Resource requests** defined in Pods
3. **Stable traffic patterns** (HPA works best with sustained load, not spikes)

## Limitations of HPA

- Can't predict traffic spikes (only reacts to current metrics)
- Needs time to scale (Pods take time to start)
- Works best with stateless Pods (Deployments, not StatefulSets)
- Requires metrics infrastructure (Metrics Server)


## Commands Summary

```bash
# Create HPA
kubectl apply -f hpa.yaml

# List HPAs
kubectl get hpa

# Detailed HPA info
kubectl describe hpa hpa-name

# Watch HPA in real-time
kubectl get hpa -w

# Delete HPA (Deployment stays at current replicas)
kubectl delete hpa hpa-name

# Edit HPA
kubectl edit hpa hpa-name
```

## High-Level Understanding

**What:** Kubernetes automatically scales Pods based on metrics
**How:** HPA watches CPU/memory, scales Deployment up/down
**Why:** Handle traffic spikes without manual intervention
**When:** When you have variable traffic patterns
**Requirements:** Metrics Server, resource requests, appropriate thresholds

---

**Progress: 49/90 days complete**
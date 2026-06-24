# Day 50: Vertical Pod Autoscaler (VPA)

## HPA vs VPA (Quick Comparison)

### HPA (Horizontal Pod Autoscaler)
- Adds more Pods
- Increases replicas (2 → 3 → 5)
- Spreads load across containers
- Good for stateless apps (web servers)

### VPA (Vertical Pod Autoscaler)
- Makes existing Pods bigger
- Increases CPU/memory requests
- Gives more resources to same Pod
- Good for stateful apps (databases, single-instance services)

**Horizontal** = scale out (more instances)
**Vertical** = scale up (bigger instance)

## How VPA Works
VPA monitors Pod resource usage

↓

Tracks actual CPU/memory consumption

↓

Compares to resource requests

↓

If Pod consistently uses more than requested:

Recommend increasing requests/limits

↓

Pod recreated with new resource values

↓

Pod uses more CPU/memory now

VPA doesn't scale instantly. It:
1. Observes for a period
2. Makes recommendations
3. Recreates Pods with new resources

## VPA Modes

### Recommendation Only
```yaml
updatePolicy:
  updateMode: "off"
```
VPA calculates recommendations but doesn't apply them. Humans decide.

### Auto-Update
```yaml
updatePolicy:
  updateMode: "Auto"
```
VPA automatically recreates Pods with new resource values.

### Recreate
```yaml
updatePolicy:
  updateMode: "Recreate"
```
Recreates Pods when resource changes needed.

## Creating a VPA

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: app-vpa
spec:
  targetRef:
    apiVersion: "apps/v1"
    kind: Deployment
    name: my-app
  updatePolicy:
    updateMode: "Auto"
  resourcePolicy:
    containerPolicies:
    - containerName: "*"
      minAllowed:
        cpu: 100m
        memory: 128Mi
      maxAllowed:
        cpu: 4
        memory: 4Gi
```

What this does:
- Watch `my-app` Deployment
- Recommend resource values between min/max
- Auto-update Pods when recommendations available
- Container can use between 100m-4 CPU, 128Mi-4Gi memory

## When to Use VPA

### Use VPA When:
- Stateful applications (databases, message queues)
- Single-instance services (need to scale the one instance)
- Unknown resource requirements (VPA figures them out)
- Overhead matters (fewer, bigger Pods is more efficient)

### Don't Use VPA When:
- Stateless applications (use HPA instead)
- Need horizontal scaling (HPA is better)
- Pod downtime is unacceptable (VPA recreates Pods)
- Unknown requests/limits (VPA needs baseline)



### Manual Right-Sizing

Deploy with guess (cpu: 500m, memory: 1Gi)
Monitor for days/weeks
See actual usage (cpu: 2000m, memory: 3Gi)
Manually update resource requests
Redeploy


Time-consuming, error-prone.

### With VPA

Deploy with baseline (cpu: 100m, memory: 128Mi)
VPA observes for ~7 days
VPA recommends correct values (cpu: 2000m, memory: 3Gi)
VPA applies them automatically
Done


Automatic, learns from actual usage.

## Viewing VPA Status

```bash
# List VPAs
kubectl get vpa

# Detailed info
kubectl describe vpa app-vpa
# Shows recommendations, current resources, status

# Check recommendations
kubectl get vpa app-vpa -o yaml | grep -A 20 "recommendation:"
```

## VPA Events

VPA logs Pod updates in events:

```bash
kubectl describe pod pod-name
# Events:
#   VPA updated resource requests for container [containerName]
#   From: cpu: 500m, memory: 1Gi
#   To: cpu: 2000m, memory: 3Gi
```

## Limitations of VPA

1. **Pod recreation** - Pod must restart for changes to take effect (causes brief downtime)
2. **No prediction** - Reacts to past usage, doesn't predict future spikes
3. **Requires history** - Needs data to make recommendations (7+ days)
4. **Conflicts with HPA** - Don't use both on same Deployment (they fight)
5. **Memory calculations are tricky** - Memory usage harder to predict than CPU

## Use HPA + VPA Together?

**No.** They conflict.

HPA scales number of Pods.
VPA scales size of Pods.

Using both causes problems:
- HPA scales up Pods
- VPA changes resource requests
- They interfere with each other

**Strategy:**
- Use HPA for stateless apps (web servers, APIs)
- Use VPA for stateful apps (databases, caches)
- Don't mix

## Installation

VPA is not built-in. Install from GitHub:

```bash
git clone https://github.com/kubernetes/autoscaler.git
cd autoscaler/vertical-pod-autoscaler
./hack/vpa-up.sh
```

Creates three components:
- `vpa-recommender` (makes recommendations)
- `vpa-updater` (applies updates)
- `vpa-admission-controller` (validates configs)

## High-Level Understanding

**What:** Kubernetes automatically adjusts Pod resource requests/limits
**How:** Monitors actual usage, recommends changes, applies them
**Why:** Right-size resources (save cost, improve performance)
**When:** Stateful apps, single-instance services
**Not:** Stateless apps (use HPA instead)

---

**Progress: 50/90 days complete**

**Video Created:** Kubernetes for Dummies (teaching reinforces learning)


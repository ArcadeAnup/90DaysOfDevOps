# Day 33: Kubernetes Services and Load Balancing

## The Problem Services Solve

### Without Services
Application needs to talk to Pods
↓
Hardcodes Pod IP (e.g., 10.244.0.5)
↓
Pod dies (replaced with new IP 10.244.0.7)
↓
Application can't find it → breaks

### With Services
Application talks to Service IP (e.g., 10.96.0.5)
↓
Service selects Pods with matching labels
↓
Pod dies and respawns (new IP)
↓
Service automatically routes to new Pod
↓
Application never knows anything changed

Services provide **stable endpoints** for **unstable Pods**.

## What is a Service?

A Service is an abstraction that:
1. Groups Pods using labels
2. Provides stable IP and DNS name
3. Load balances traffic between Pods
4. Exposes applications (internally or externally)

Think of it as a load balancer pointing to a group of Pods.

## Service Types

### ClusterIP (Default)
- Internal cluster communication only
- Stable IP only accessible within cluster
- Use case: Backend services

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  type: ClusterIP
  selector:
    app: backend
  ports:
  - port: 3000
    targetPort: 3000
```

### NodePort
- Exposes service on a port on every node
- Accessible externally via `NodeIP:NodePort`
- Good for development, not production

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  type: NodePort
  selector:
    app: frontend
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 30080        # Range 30000-32767
```

Access via: `<node-ip>:30080`

### LoadBalancer
- Creates external load balancer (cloud provider)
- Assigns external IP
- Production-grade

```yaml
apiVersion: v1
kind: Service
metadata:
  name: api-service
spec:
  type: LoadBalancer
  selector:
    app: api
  ports:
  - port: 80
    targetPort: 8000
```

Cloud provider provisions external IP automatically.

## Labels and Selectors

Services find Pods using labels and selectors.

### Adding Labels to Pods

In Deployment YAML:
```yaml
spec:
  template:
    metadata:
      labels:
        app: nginx              # Label the Pod
        version: v1
```

### Service Selector

In Service YAML:
```yaml
spec:
  selector:
    app: nginx                  # Select Pods with this label
```

Service finds all Pods with `app: nginx` label and routes traffic to them.

### Label Commands

```bash
# See Pod labels
kubectl get pods --show-labels

# Label a Pod
kubectl label pod pod-name app=nginx

# Filter by label
kubectl get pods -l app=nginx

# Remove label
kubectl label pod pod-name app-
```

## Creating a Service (Step by Step)

### 1. Ensure Pods Have Labels

```bash
# Create Deployment (Pods automatically get labels from template)
kubectl apply -f deployment.yaml

# Verify labels
kubectl get pods --show-labels
```

### 2. Create Service YAML

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  labels:
    app: nginx-service
spec:
  type: NodePort
  selector:
    app: nginx                  # Must match Pod label
  ports:
  - protocol: TCP
    port: 80                    # Service port
    targetPort: 80              # Pod container port
    nodePort: 30008             # External port (30000-32767)
```

### 3. Dry Run (Test Without Creating)

```bash
kubectl apply -f service.yaml --dry-run=client -o yaml

# Preview what will be created
# If output looks good, apply for real
```

**Why dry run matters:** Catch errors before deploying. See exactly what Kubernetes will create.

### 4. Deploy Service

```bash
kubectl apply -f service.yaml
```

### 5. Verify Service

```bash
# See Service
kubectl get services

# Get Service details
kubectl describe service nginx-service

# Output shows:
# - Cluster IP (internal)
# - External endpoints (if applicable)
# - Selector labels
# - Port mappings
```

## Port Mapping in Services

```yaml
ports:
- protocol: TCP
  port: 80              # Port on Service (what applications use)
    targetPort: 80      # Port on Pod container (where app listens)
    nodePort: 30008     # Port on node (external access)
```

**Flow:**
External User → Node:30008 → Service IP:80 → Pod Container:80


```bash
# Get KinD node IP
kubectl get nodes -o wide

# Get Service NodePort
kubectl get service nginx-service

# Access via: <node-ip>:<nodeport>
# Example: 172.18.0.2:30008
```


## Load Balancing in Action

```bash
# Get Pod endpoints
kubectl get endpoints nginx-service

# Output shows all Pods the Service sends traffic to:
# NAME             ENDPOINTS                            AGE
# nginx-service    10.244.0.3:80,10.244.0.4:80,...     2m
```

Service automatically routes traffic across all Pods.

## Practical Example: Full Stack

### 1. Create Deployment (4 Nginx Pods)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 4
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx              # Important: label for Service
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

### 2. Create Service
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app: nginx                  # Matches Deployment labels
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30008
```

### 3. Deploy Both
```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

### 4. Test
```bash
# Port forward
kubectl port-forward service/nginx-service 8080:80

# Access browser
# localhost:8080 → Service → One of 4 Pods → Nginx running ✅
```

If you refresh page multiple times, you might see different Pod serving (load balancing working).

## Common Service Commands

```bash
# List Services
kubectl get services
kubectl get svc                          # Short form

# Service details
kubectl describe service service-name

# Service endpoints (Pods it's routing to)
kubectl get endpoints

# Port forward
kubectl port-forward service/name 8080:80

# Delete Service
kubectl delete service service-name

# Expose Deployment as Service (imperative)
kubectl expose deployment nginx-deployment \
  --type=NodePort \
  --port=80
```

## Selectors vs Ports (Easy to Confuse)

```yaml
spec:
  selector:
    app: nginx          # This selects which Pods to route to

  ports:
  - port: 80           # This is the Service IP port
    targetPort: 80     # This is the Pod container port
```

**Selector** = "which Pods"
**Ports** = "how to route traffic"

## Key Concepts

1. **Services provide stable endpoints** for ephemeral Pods
2. **Labels connect Services to Pods** (selector finds them)
3. **Load balancing is automatic** across selected Pods
4. **DNS is built in** (use service name instead of IP)
5. **Dry run catches errors** before deploying
6. **Port forwarding accesses services locally** (development)
7. **NodePort exposes externally** (simple, not production)

## Real-World Pattern
Deployment (manages Pods) + Service (exposes Pods) =
Complete application unit

You almost never deploy Pods without Services in real applications.


**Progress: 33/90 days complete**

**Architecture Understanding:** Deployment + Service = how real Kubernetes applications work. Rest is refinement.
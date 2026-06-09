# Day 35: End-to-End Django → Docker → Kubernetes Deployment

## The Complete Workflow
Source Code → Docker Image → Registry → Kubernetes → Running App


## Step 1: Containerize Django App

### Dockerfile

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
```

### Build Image

```bash
docker build -t my-django-app .
```

### Test Locally

```bash
docker run -p 8000:8000 my-django-app

# Access: http://localhost:8000 ✅
```

## Step 2: Push to DockerHub

### Tag Image

```bash
docker tag my-django-app <dockerhub-username>/my-django-app:latest
```

### Login to DockerHub

```bash
docker login
```

### Push

```bash
docker push <dockerhub-username>/my-django-app:latest
```

Image now publicly available: `https://hub.docker.com/r/<username>/my-django-app`

## Step 3: Create Kubernetes Namespace

Isolate resources (good practice for organization).

```bash
kubectl create namespace django-app
```

Verify:
```bash
kubectl get namespaces
```

## Step 4: Create Kubernetes Deployment

### Deployment YAML

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: django-deployment
  namespace: django-app
  labels:
    app: django
spec:
  replicas: 3                    # 3 copies running
  selector:
    matchLabels:
      app: django
  template:
    metadata:
      labels:
        app: django
    spec:
      containers:
      - name: django
        image: <username>/my-django-app:latest    # From DockerHub
        ports:
        - containerPort: 8000
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
```

### Deploy

```bash
kubectl apply -f deployment.yaml
```

### Verify Deployment

```bash
# See Deployment
kubectl get deployment -n django-app

# See Pods
kubectl get pods -n django-app

# See details
kubectl describe deployment django-deployment -n django-app

# Pod logs
kubectl logs <pod-name> -n django-app
```

**Output:**
NAME                               READY   STATUS    RESTARTS   AGE
django-deployment-7d9f8f7f9f-abc   1/1     Running   0          2m
django-deployment-7d9f8f7f9f-def   1/1     Running   0          2m
django-deployment-7d9f8f7f9f-ghi   1/1     Running   0          2m

3 Pods running, load balanced.

## Step 5: Create Kubernetes Service

### Service YAML

```yaml
apiVersion: v1
kind: Service
metadata:
  name: django-service
  namespace: django-app
  labels:
    app: django
spec:
  type: NodePort                 # Expose externally
  selector:
    app: django                  # Find Pods with this label
  ports:
  - protocol: TCP
    port: 8000                   # Service port
    targetPort: 8000             # Pod container port
    nodePort: 30080              # External node port
```

### Deploy Service

```bash
kubectl apply -f service.yaml
```

### Verify Service

```bash
# See Service
kubectl get service -n django-app

# See endpoints (which Pods it's routing to)
kubectl get endpoints -n django-app

# Output:
# NAME             ENDPOINTS                           AGE
# django-service   10.244.0.3:8000,10.244.0.4:8000,... 1m
```

Service is routing to all 3 Pods.

## Step 6: Access the App

### Option 1: Port Forward (Development)

```bash
kubectl port-forward service/django-service 8000:8000 -n django-app
```

Access: `http://localhost:8000` ✅

### Option 2: NodePort (Production-like)

```bash
# Get node IP
kubectl get nodes -o wide

# Access via: <node-ip>:30080
```

## Load Balancing in Action

Service automatically distributes traffic across 3 Pods.

```bash
# Check which Pod served request (Django logs will show)
kubectl logs <pod-1> -n django-app
kubectl logs <pod-2> -n django-app
kubectl logs <pod-3> -n django-app

# Different Pods will have request logs (proving load balancing works)
```

## Self-Healing in Action

```bash
# Delete a Pod
kubectl delete pod <pod-name> -n django-app

# Watch Deployment
kubectl get pods -n django-app -w

# New Pod immediately created
# Deployment maintains 3 replicas always
```

## Complete Workflow Summary

Write Django code
↓
Create Dockerfile
↓
Build Docker image locally
↓
Test image locally
↓
Push to DockerHub
↓
Create Kubernetes Deployment YAML (references DockerHub image)
↓
Apply Deployment
↓
Create Kubernetes Service YAML
↓
Apply Service
↓
Access running app


No manual deployment steps. No clicking servers. No "works on my machine" problems.

Everything is code, everything is reproducible.

## Key Commands Reference

```bash
# Build Docker image
docker build -t app-name .

# Push to registry
docker push username/app-name

# Create namespace
kubectl create namespace app-namespace

# Apply Kubernetes resources
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

# View resources in namespace
kubectl get deployment -n app-namespace
kubectl get pods -n app-namespace
kubectl get service -n app-namespace

# Port forward
kubectl port-forward service/service-name 8000:8000 -n namespace

# Check logs
kubectl logs pod-name -n namespace

# Delete resources
kubectl delete -f deployment.yaml -n namespace
kubectl delete service service-name -n namespace
```



## Realization


Docker + Kubernetes + code = real applications running.

---

**Progress: 35/90 days complete**


# Day 44: Kubernetes Ingress with NGINX Controller on KinD

## What is Ingress? (Recap)

Ingress = Kubernetes resource that routes external HTTP/HTTPS traffic to Services based on:
- Hostnames (example.com vs api.example.com)
- URL paths (/api vs /static)
- TLS certificates

**Ingress != networking magic. It's a reverse proxy (NGINX) reading your YAML rules.**

## Architecture
Internet Traffic

↓

NGINX Ingress Controller (Pod running NGINX)

↓

Ingress Rules (YAML that controller reads)

↓

Services (which load balance to Pods)

↓

Application Pods

Controller watches for Ingress resources and updates its config.

## Install NGINX Ingress Controller on KinD

### Step 1: Create KinD Cluster

```bash
# Create cluster with extraPortMappings for HTTP/HTTPS
cat <<EOF | kind create cluster --config=-
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
EOF
```

This allows accessing Ingress on localhost:80 and localhost:443.

### Step 2: Install NGINX Ingress Controller

Using Helm (package manager for Kubernetes):

```bash
# Add Helm repo
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update

# Install NGINX Ingress Controller
helm install nginx-ingress ingress-nginx/ingress-nginx \
  --namespace ingress-nginx \
  --create-namespace \
  --set controller.service.type=NodePort \
  --set controller.service.nodePorts.http=32080 \
  --set controller.service.nodePorts.https=32443
```

### Step 3: Verify Installation

```bash
# Check NGINX Ingress Controller Pod
kubectl get pods -n ingress-nginx

# Output:
# NAME                                        READY   STATUS    RESTARTS   AGE
# nginx-ingress-ingress-nginx-controller...   1/1     Running   0          2m

# Check Service
kubectl get svc -n ingress-nginx

# Output:
# NAME                                 TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)
# nginx-ingress-ingress-nginx-controller  NodePort   10.96.0.2       <none>        80:32080/TCP,443:32443/TCP
```

NGINX controller running and listening on port 80.

## Create Test Applications

### Deployment 1: Frontend

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
        volumeMounts:
        - name: html
          mountPath: /usr/share/nginx/html
      volumes:
      - name: html
        configMap:
          name: frontend-html

---
apiVersion: v1
kind: ConfigMap
metadata:
  name: frontend-html
data:
  index.html: |
    <h1>Frontend Application</h1>
    <p>This is served via Ingress from myapp.localhost</p>

---
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  selector:
    app: frontend
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
```

### Deployment 2: API

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api
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
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
        volumeMounts:
        - name: html
          mountPath: /usr/share/nginx/html
      volumes:
      - name: html
        configMap:
          name: api-html

---
apiVersion: v1
kind: ConfigMap
metadata:
  name: api-html
data:
  index.html: |
    <h1>API Service</h1>
    <p>This is served via Ingress from api.localhost</p>

---
apiVersion: v1
kind: Service
metadata:
  name: api-service
spec:
  selector:
    app: api
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP
```

Deploy both:
```bash
kubectl apply -f frontend.yaml
kubectl apply -f api.yaml

# Verify
kubectl get deployments
kubectl get services
```

## Create Ingress Resources

### Option 1: Host-Based Routing

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: main-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: myapp.localhost
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
  - host: api.localhost
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
```

**What this does:**
- Traffic to `myapp.localhost` → frontend-service
- Traffic to `api.localhost` → api-service

### Option 2: Path-Based Routing

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: path-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: localhost
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
```

**What this does:**
- Traffic to `localhost/` → frontend-service
- Traffic to `localhost/api` → api-service

Apply Ingress:
```bash
kubectl apply -f ingress.yaml

# Verify
kubectl get ingress

# Output:
# NAME           CLASS   HOSTS                      ADDRESS        PORTS   AGE
# main-ingress   nginx   myapp.localhost,api...    <pending>       80      2m
```

## Test Ingress (Host-Based)

### Option 1: Update /etc/hosts

```bash
# Edit /etc/hosts (requires sudo on Linux/Mac)
sudo nano /etc/hosts

# Add:
127.0.0.1 myapp.localhost
127.0.0.1 api.localhost
```

Now you can access:
```bash
curl http://myapp.localhost
# Output: <h1>Frontend Application</h1>

curl http://api.localhost
# Output: <h1>API Service</h1>
```

### Option 2: Using curl with Host Header

```bash
# Tell curl which hostname you're accessing
curl -H "Host: myapp.localhost" http://localhost

# Output: <h1>Frontend Application</h1>

curl -H "Host: api.localhost" http://localhost

# Output: <h1>API Service</h1>
```

### Option 3: Port Forward (for testing)

```bash
# Port forward NGINX controller to localhost:8080
kubectl port-forward -n ingress-nginx service/nginx-ingress-ingress-nginx-controller 8080:80

# In another terminal:
curl -H "Host: myapp.localhost" http://localhost:8080
# Output: <h1>Frontend Application</h1>
```

## Test Ingress (Path-Based)

```bash
curl http://localhost/
# Output: <h1>Frontend Application</h1>

curl http://localhost/api
# Output: <h1>API Service</h1>
```

## How NGINX Controller Works

### Behind the Scenes

When you create an Ingress:

1. **NGINX Controller watches for Ingress resources**
Watch: kubectl get ingress

Sees: main-ingress created

2. **Controller reads Ingress spec**
Rules: myapp.localhost → frontend-service

api.localhost → api-service

3. **Controller generates NGINX config**
```nginx
   server {
     listen 80;
     server_name myapp.localhost;
     location / {
       proxy_pass http://frontend-service:80;
     }
   }
   
   server {
     listen 80;
     server_name api.localhost;
     location / {
       proxy_pass http://api-service:80;
     }
   }
```

4. **Controller reloads NGINX**
nginx -s reload

5. **NGINX routes traffic**
Request: GET http://myapp.localhost/

↓

NGINX sees server_name matches

↓

proxy_pass → frontend-service:80

↓

Service load balances to frontend Pods

This happens automatically. You just write YAML, controller handles the rest.

## View Controller Logs

```bash
# See what controller is doing
kubectl logs -n ingress-nginx \
  deployment/nginx-ingress-ingress-nginx-controller -f

# Output shows:
# controller.go:300] ingress main-ingress created
# controller.go:310] starting NGINX reload
# ... (NGINX reloads)
# controller.go:320] NGINX reloaded successfully
```

## Complete Hands-On Example

```bash
# 1. Create cluster with port mappings
kind create cluster --config=kind-config.yaml

# 2. Install NGINX controller
helm install nginx-ingress ingress-nginx/ingress-nginx \
  --namespace ingress-nginx --create-namespace

# 3. Create deployments and services
kubectl apply -f frontend.yaml
kubectl apply -f api.yaml

# 4. Create Ingress
kubectl apply -f ingress.yaml

# 5. Test
# Update /etc/hosts with myapp.localhost and api.localhost
curl http://myapp.localhost
curl http://api.localhost
```

Both services accessible via Ingress. Single entry point. Clean routing.

## Key Insights from Day 44

1. **Ingress is a reverse proxy** - NGINX controller implements it
2. **Host-based routing** - Different apps on different domains
3. **Path-based routing** - Different services on different paths
4. **Controller watches YAML** - Automatic NGINX reconfiguration
5. **Load balancing is built in** - Service handles distribution to Pods
6. **Production-ready** - This is how real Kubernetes exposes apps

## Ingress vs NodePort vs LoadBalancer

| | NodePort | LoadBalancer | Ingress |
|---|----------|--------------|---------|
| **Port** | 30000-32767 | Any | 80, 443 |
| **Hostnames** | No | No | Yes |
| **Paths** | No | No | Yes |
| **TLS** | No | No | Yes |
| **Multiple services** | One per LB | One per LB | Many |
| **Cost** | Free | Expensive (cloud LB) | One controller |

**Development:** NodePort or port-forward
**Production:** Ingress with TLS certificates


apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: secure-ingress
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - myapp.localhost
    secretName: tls-secret
  rules:
  - host: myapp.localhost
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
```

HTTPS automatically configured.

## Commands Summary

```bash
# Install NGINX Ingress Controller
helm install nginx-ingress ingress-nginx/ingress-nginx \
  --namespace ingress-nginx --create-namespace

# Create Ingress
kubectl apply -f ingress.yaml

# View Ingress
kubectl get ingress
kubectl describe ingress ingress-name

# View controller logs
kubectl logs -n ingress-nginx deployment/nginx-ingress-ingress-nginx-controller -f

# Port forward for testing
kubectl port-forward -n ingress-nginx svc/nginx-ingress-ingress-nginx-controller 8080:80

# Delete Ingress
kubectl delete ingress ingress-name
```

## What's Next

1. **TLS Certificates** - cert-manager for automatic Let's Encrypt
2. **Multiple Ingress Controllers** - Route different traffic differently
3. **Ingress with Auth** - Basic auth, OAuth2
4. **Advanced routing** - Regex patterns, regex rewrites
5. **Service Mesh** - Istio (advanced traffic management)



**Progress: 44/90 days complete**

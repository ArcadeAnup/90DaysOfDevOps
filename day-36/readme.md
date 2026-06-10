# Day 36: Kubernetes Ingress (Theory)

## The Problem with NodePort and LoadBalancer

### NodePort
- Exposes service on random high-numbered port (30000-32767)
- Not production-friendly
- Can't use standard ports (80, 443)
- URL looks like: `198.51.100.5:30847` (ugly)

### LoadBalancer
- Cloud provider provisions external load balancer
- Gets external IP
- Better than NodePort, but expensive
- One LoadBalancer per Service

### What We Really Need
- Single entry point for all services
- Multiple services accessible via different domains/paths
- Standard ports (80, 443)
- TLS/HTTPS support
- User-friendly URLs

**Enter Ingress.**

## What is Kubernetes Ingress?

Ingress is a Kubernetes resource that:
1. Routes external HTTP/HTTPS traffic
2. Based on hostnames and URL paths
3. To internal Kubernetes Services
4. Provides a single IP address for all services

Think of it as a reverse proxy for Kubernetes.

## How It Works
User on Internet
↓
DNS resolves myapp.com → Ingress Controller IP
↓
Ingress Controller receives HTTP request
↓
Looks at URL/hostname
↓
Ingress Rules define routing:

myapp.com → frontend Service
api.myapp.com → backend Service
↓
Routes to appropriate Service
↓
Service load balances to Pods
↓
Container responds


## Ingress vs Service

### Service (Kubernetes)
- Internal networking layer
- Exposes Pods to other Pods or externally
- Layer 4 (TCP/UDP)

### Ingress (Kubernetes)
- Layer 7 (HTTP/HTTPS)
- Routes based on URLs, hostnames
- Requires external Ingress Controller
- Manages external access to services

**Both together:**
Services handle Kubernetes networking, Ingress handles external routing.

## Ingress Components

### 1. Ingress Resource
YAML file defining routing rules.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
spec:
  rules:
  - host: myapp.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
  - host: api.myapp.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: backend-service
            port:
              number: 8000
```

### 2. Ingress Controller
Actual component that reads Ingress resources and routes traffic.

**Popular Controllers:**
- NGINX Ingress Controller (most common)
- AWS ALB Ingress Controller
- Google Cloud Load Balancing
- Istio (advanced, service mesh)

The controller watches for Ingress resources and configures itself (e.g., NGINX updates config and reloads).

### 3. External Load Balancer (optional)
Cloud provider's load balancer that directs traffic to Ingress Controller.

## Routing Types

### Host-Based Routing
Route based on hostname:

```yaml
rules:
- host: frontend.myapp.com
  http:
    paths:
    - path: /
      backend:
        service:
          name: frontend-service
          port:
            number: 80

- host: api.myapp.com
  http:
    paths:
    - path: /
      backend:
        service:
          name: api-service
          port:
            number: 8000
```

Users access: `frontend.myapp.com` or `api.myapp.com`

### Path-Based Routing
Route based on URL path:

```yaml
rules:
- host: myapp.com
  http:
    paths:
    - path: /api
      backend:
        service:
          name: api-service
          port:
            number: 8000
    - path: /
      backend:
        service:
          name: frontend-service
          port:
            number: 80
```

Users access: `myapp.com/` (frontend) or `myapp.com/api` (backend)

### Combination
Both host and path based:

```yaml
rules:
- host: myapp.com
  http:
    paths:
    - path: /api
      backend: api-service
    - path: /
      backend: frontend-service

- host: admin.myapp.com
  http:
    paths:
    - path: /
      backend: admin-service
```

## TLS/HTTPS with Ingress

Ingress can terminate TLS (SSL/HTTPS):

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress-tls
spec:
  tls:
  - hosts:
    - myapp.com
    secretName: myapp-tls-secret
  rules:
  - host: myapp.com
    http:
      paths:
      - path: /
        backend:
          service:
            name: frontend-service
            port:
              number: 80
```

TLS certificate stored in Kubernetes Secret (`myapp-tls-secret`). Ingress Controller uses it to handle HTTPS.

Users access: `https://myapp.com` (secure)

## Installation (High Level)

Ingress is a Kubernetes resource, but it needs a controller to work.

### Install NGINX Ingress Controller

```bash
# Using Helm (package manager for Kubernetes)
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm install nginx-ingress ingress-nginx/ingress-nginx
```

This deploys the NGINX Ingress Controller to your cluster.

### Get External IP

```bash
kubectl get service nginx-ingress-controller

# Output:
# NAME                              TYPE           CLUSTER-IP     EXTERNAL-IP
# nginx-ingress-ingress-nginx       LoadBalancer   10.0.0.5       198.51.100.5
```

External IP is where traffic comes in.

### Point DNS to External IP

Update DNS records:
myapp.com  A  198.51.100.5
api.myapp.com  A  198.51.100.5

Both domains point to Ingress Controller IP.

## Ingress vs LoadBalancer Service

### LoadBalancer Service
Internet → Cloud Load Balancer (for THIS service) → Service → Pods

Expensive: one load balancer per service.

### Ingress
Internet → Single Ingress Controller → Multiple Services → Pods

Efficient: one entry point for all services.

## Real-World Example

**Microservices cluster:**
- Frontend (React)
- API (Django)
- Admin panel (Next.js)
- Worker (background jobs, no external access)

### With LoadBalancer Services
Internet
├→ Load Balancer 1 → frontend Service → frontend Pods
├→ Load Balancer 2 → api Service → api Pods
├→ Load Balancer 3 → admin Service → admin Pods

3 separate load balancers = expensive.

### With Ingress
Internet → Ingress Controller (NGINX)
├→ frontend.myapp.com → frontend Service → frontend Pods
├→ api.myapp.com → api Service → api Pods
├→ admin.myapp.com → admin Service → admin Pods

1 entry point, multiple services. Much cheaper.

## Key Concepts

1. **Ingress** = Routing rules (YAML)
2. **Ingress Controller** = Actual reverse proxy (NGINX, AWS ALB, etc.)
3. **Service** = Routes within cluster (Layer 4)
4. **Ingress** = Routes from internet (Layer 7)
5. **Host-based** = Multiple domains
6. **Path-based** = Multiple paths
7. **TLS** = HTTPS termination

## Why Ingress Matters

- **Production-ready** - Proper DNS and HTTPS
- **Cost-efficient** - One entry point for multiple services
- **Scaling** - Add new services without changing external infrastructure
- **High availability** - Ingress Controller can be replicated
- **Security** - TLS termination, DDoS protection (depending on controller)

## Common Ingress Controllers

### NGINX
- Most popular
- Open source
- Feature-rich

### AWS ALB
- AWS-specific
- Deep integration with AWS
- Automatic certificate management

### GCP Cloud Load Balancer
- Google-specific
- Integrates with GCP ecosystem

### Traefik
- Modern, dynamic
- Auto-discovers services
- Built-in dashboard

### Istio
- Advanced (service mesh)
- Traffic management, security, observability
- Overkill for simple use cases

## Ingress Limitations

1. **Requires Controller** - Ingress resource alone does nothing
2. **No TCP/UDP** - HTTP/HTTPS only (use Service LoadBalancer for TCP)
3. **Not a service mesh** - Doesn't provide advanced routing, retry logic, circuit breaking
4. **DNS management** - You still need to manage DNS records

## What's Next After Ingress

Once you understand Ingress, you're ready for:
1. **Certificates** - Let's Encrypt, cert-manager
2. **Service Mesh** - Istio, Linkerd (advanced)
3. **Network Policies** - Security between Pods
4. **Load Balancing** - Advanced algorithms

## Real-World Architecture
Users on Internet
↓
DNS (myapp.com → Ingress Controller IP)
↓
Ingress Controller (NGINX) running on Kubernetes
↓
Ingress Rules (routing by hostname/path)
↓
Services (load balance across Pods)
↓
Pods (containers running applications)
↓
Databases (external or StatefulSet)

This is the actual production architecture for most Kubernetes deployments.

## Takeaway

Ingress is the production networking layer for Kubernetes. Without it, you're stuck with NodePort (ugly) or LoadBalancer per service (expensive).

Understanding Ingress means understanding how real Kubernetes applications are exposed to the internet.

---

**Progress: 36/90 days complete**

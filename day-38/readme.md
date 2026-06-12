
## ConfigMaps vs Secrets

### ConfigMaps
- **Purpose:** Non-sensitive configuration
- **Data:** Plain text, up to 1MB
- **Use case:** Database host, API endpoints, feature flags
- **Encoding:** Plain (not encoded)
- **Security:** Anyone with Kubernetes access can read

### Secrets
- **Purpose:** Sensitive data
- **Data:** Base64-encoded by default, up to 1MB
- **Use case:** Passwords, API keys, tokens, certificates
- **Encoding:** Base64 (not encryption, just encoding)
- **Security:** Better than hardcoding, but enable encryption at rest for production

## ConfigMaps

### Creating ConfigMaps

#### Method 1: YAML File

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: default
data:
  DATABASE_HOST: "postgres.example.com"
  DATABASE_PORT: "5432"
  API_ENDPOINT: "https://api.example.com"
  LOG_LEVEL: "info"
```

Create:
```bash
kubectl apply -f configmap.yaml
```

#### Method 2: From File

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
data:
  nginx.conf: |
    server {
      listen 80;
      server_name example.com;
      location / {
        proxy_pass http://backend;
      }
    }
```

#### Method 3: Imperative (Command Line)

```bash
kubectl create configmap app-config \
  --from-literal=DATABASE_HOST=postgres.example.com \
  --from-literal=DATABASE_PORT=5432
```

### Using ConfigMaps in Pods

#### As Environment Variables

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: app
    image: my-app:latest
    env:
    - name: DATABASE_HOST
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: DATABASE_HOST
    - name: DATABASE_PORT
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: DATABASE_PORT
```

Inside container:
```bash
echo $DATABASE_HOST
# Output: postgres.example.com
```

#### Mount as File

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:latest
    volumeMounts:
    - name: config-volume
      mountPath: /etc/nginx/conf.d
  volumes:
  - name: config-volume
    configMap:
      name: nginx-config
```

Inside container:
```bash
ls /etc/nginx/conf.d/
# Output: nginx.conf
cat /etc/nginx/conf.d/nginx.conf
# Shows content from ConfigMap
```

### ConfigMap in Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: app
        image: my-app:latest
        envFrom:
        - configMapRef:
            name: app-config     # Load all ConfigMap keys as env vars
      volumes:
      - name: config
        configMap:
          name: app-config
```

All ConfigMap keys automatically become environment variables.

## Secrets

### Creating Secrets

#### Method 1: YAML File (Base64 Encoded)

```bash
# Encode passwords in base64
echo -n "mysecretpassword" | base64
# Output: bXlzZWNyZXRwYXNzd29yZA==

echo -n "sk-1234567890abcdef" | base64
# Output: c2stMTIzNDU2Nzg5MGFiY2RlZg==
```

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
type: Opaque
data:
  DATABASE_PASSWORD: bXlzZWNyZXRwYXNzd29yZA==
  API_KEY: c2stMTIzNDU2Nzg5MGFiY2RlZg==
```

Create:
```bash
kubectl apply -f secret.yaml
```

#### Method 2: Imperative (Command Line)

```bash
kubectl create secret generic app-secrets \
  --from-literal=DATABASE_PASSWORD=mysecretpassword \
  --from-literal=API_KEY=sk-1234567890abcdef
```

#### Method 3: From Files

```bash
# For TLS certificates
kubectl create secret tls tls-secret \
  --cert=path/to/cert \
  --key=path/to/key
```

### Secret Types

**Opaque (default):**
```yaml
type: Opaque
data:
  password: base64-encoded-password
```

**Docker Registry (pull private images):**
```yaml
type: kubernetes.io/dockercfg
data:
  .dockercfg: base64-encoded-docker-config
```

**TLS/SSL:**
```yaml
type: kubernetes.io/tls
data:
  tls.crt: base64-cert
  tls.key: base64-key
```

**Basic Auth:**
```yaml
type: kubernetes.io/basic-auth
data:
  username: base64-username
  password: base64-password
```

### Using Secrets in Pods

#### As Environment Variables

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: db-pod
spec:
  containers:
  - name: db
    image: postgres:latest
    env:
    - name: POSTGRES_PASSWORD
      valueFrom:
        secretKeyRef:
          name: app-secrets
          key: DATABASE_PASSWORD
    - name: API_KEY
      valueFrom:
        secretKeyRef:
          name: app-secrets
          key: API_KEY
```

#### Mount as File

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  containers:
  - name: app
    image: my-app:latest
    volumeMounts:
    - name: secret-volume
      mountPath: /etc/secrets
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: app-secrets
```

Inside container:
```bash
cat /etc/secrets/DATABASE_PASSWORD
# Output: mysecretpassword (decoded automatically)
```

### Secret in Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: app
        image: my-app:latest
        envFrom:
        - secretRef:
            name: app-secrets    # Load all Secret keys as env vars
```

## Real-World Pattern

### Dockerfile (Generic, No Secrets)
```dockerfile
FROM python:3.11
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["python", "app.py"]
```

### ConfigMap (Non-sensitive config)
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: db-config
data:
  DATABASE_HOST: postgres.example.com
  DATABASE_PORT: "5432"
  DATABASE_NAME: myapp_db
  LOG_LEVEL: info
```

### Secret (Sensitive data)
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secrets
type: Opaque
data:
  DATABASE_USER: YXBwdXNlcg==           # appuser
  DATABASE_PASSWORD: c2VjcmV0cGFzcw==  # secretpass
  API_KEY: c2stYWJjZGVm                # sk-abcdef
```

### Deployment (Uses both)
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deployment
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: app
        image: my-app:latest
        envFrom:
        - configMapRef:
            name: db-config
        - secretRef:
            name: db-secrets
```

Same image, different environments. Change ConfigMap or Secret → Pods automatically pick up new values (depends on update strategy).

## Updating Configs

### Update ConfigMap

```bash
kubectl edit configmap app-config
# Opens editor, change values
```

Or:
```bash
kubectl patch configmap app-config -p \
  '{"data":{"LOG_LEVEL":"debug"}}'
```

### Update Secret

```bash
# Delete old secret
kubectl delete secret app-secrets

# Create new secret
kubectl create secret generic app-secrets \
  --from-literal=DATABASE_PASSWORD=newpassword
```

**Note:** Updating ConfigMap/Secret doesn't automatically restart Pods. Pods need to be restarted to pick up changes (or use sidecar that watches for updates).

## Important Notes

### Base64 is Not Encryption
```bash
echo "mysecretpassword" | base64
# Output: bXlzZWNyZXRwYXNzd29yZA==

echo "bXlzZWNyZXRwYXNzd29yZA==" | base64 -d
# Output: mysecretpassword
```

Base64 is just encoding. Anyone with Kubernetes access can decode it.

**For production:** Enable encryption at rest in etcd (Kubernetes database).

### Secrets Are Not Encrypted by Default

```bash
# Retrieve Secret in plain text
kubectl get secret app-secrets -o yaml

# Shows base64-encoded data (easily decoded)
```

Best practices:
- Enable encryption at rest in etcd
- Use RBAC (role-based access control)
- Audit logs for Secret access
- Consider external secret management (HashiCorp Vault, AWS Secrets Manager)

## ConfigMap and Secret Limits

- Maximum size: 1MB per ConfigMap/Secret
- For larger configs: Mount as volumes, store in external system
- For very sensitive data: Use HashiCorp Vault, AWS Secrets Manager

## Commands Summary

```bash
# ConfigMaps
kubectl create configmap <name> --from-literal=key=value
kubectl get configmap
kubectl describe configmap <name>
kubectl edit configmap <name>
kubectl delete configmap <name>

# Secrets
kubectl create secret generic <name> --from-literal=key=value
kubectl get secret
kubectl describe secret <name>
kubectl get secret <name> -o yaml
kubectl delete secret <name>

# View Secret data (base64 decoded)
kubectl get secret <name> -o jsonpath='{.data.key}' | base64 -d
```

## Key Concepts

1. **ConfigMaps** = Non-sensitive configuration
2. **Secrets** = Sensitive data (base64 encoded by default)
3. **Decouple code from config** = Same image, different environments
4. **Environment variables** = Load ConfigMap/Secret as env vars
5. **Volume mounts** = Load ConfigMap/Secret as files
6. **Base64 is not encryption** = Enable encryption at rest for production
7. **RBAC** = Control who can access Secrets

## Real-World Scenario

**Development:**
- ConfigMap with local database host
- Secret with local database password

**Staging:**
- ConfigMap with staging database host
- Secret with staging database password

**Production:**
- ConfigMap with production database host
- Secret with production database password (from Vault)

Same Docker image deployed three times. Configuration differs by environment.



**Progress: 38/90 days complete**

**Realization:** ConfigMaps and Secrets are how you go from toy projects to real production systems. Same code, different configurations. This is the foundation of GitOps.
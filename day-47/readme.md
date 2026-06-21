# Day 47: Kubernetes Persistent Volumes and Claims (High Level)

## The Problem

Pods are ephemeral. When a Pod dies:
- Container data disappears
- Logs vanish
- Database data lost
- State is gone

You can't run stateful applications (databases, caches) without persistent storage.

## The Solution: Persistent Volumes (PV) and Persistent Volume Claims (PVC)

### Two Concepts, Clear Separation

**Persistent Volume (PV):**
- Storage resource in cluster
- Created by administrator
- Exists independent of Pods
- Defines: storage type, size, access mode

**Persistent Volume Claim (PVC):**
- Request for storage
- Created by developer/application
- Says: "I need 10GB of storage"
- Kubernetes matches PVC to available PV

### The Flow
Admin creates PV (storage available)

↓

Developer creates PVC (request storage)

↓

Kubernetes matches PVC to PV (binding)

↓

Pod references PVC (uses storage)

↓

Pod dies → Data persists

↓

New Pod claims same PVC → Accesses same data

**Storage decoupled from Pods.** Pod-agnostic.

## PV and PVC Analogy

**Real world:**
- PV = Apartment building (exists)
- PVC = Rental application (request for apartment)
- Binding = Tenant moves into apartment
- Pod = Tenant living in apartment

Tenant leaves, apartment remains. Next tenant moves in, finds same place.

## Creating PV and PVC

### Persistent Volume (Administrator Creates)

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-storage
spec:
  capacity:
    storage: 10Gi          # 10 gigabytes
  accessModes:
    - ReadWriteOnce        # Single Pod can read/write
  persistentVolumeReclaimPolicy: Retain  # Keep data when PVC deleted
  hostPath:
    path: /data            # Local node storage
```

This PV says: "I have 10GB of storage available at /data"



## Access Modes

**ReadWriteOnce (RWO):**
- One Pod can read and write
- Pod owns the volume exclusively
- Good for databases

**ReadOnlyMany (ROMany):**
- Multiple Pods can read
- No writes allowed
- Good for config/data sharing

**ReadWriteMany (RWMany):**
- Multiple Pods can read and write
- Requires network storage (NFS, etc.)
- Good for shared data

#

```

When PVC created, Storage Class automatically provisions a PV.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-storage
spec:
  storageClassName: fast-storage  # Auto-creates PV
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

## Database Example: PostgreSQL

```yaml
# Storage Class (dynamic provisioning)
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: postgres-storage
provisioner: kubernetes.io/azure-disk

---

# PVC (request storage)
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
spec:
  storageClassName: postgres-storage
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi

---

# Deployment (uses PVC)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
spec:
  replicas: 1
  template:
    spec:
      containers:
      - name: postgres
        image: postgres:14
        env:
        - name: POSTGRES_PASSWORD
          value: "password"
        volumeMounts:
        - name: postgres-storage
          mountPath: /var/lib/postgresql/data
      volumes:
      - name: postgres-storage
        persistentVolumeClaim:
          claimName: postgres-pvc
```

Deploy this: PostgreSQL gets 20GB of storage. Data persists across Pod restarts and node failures (depending on storage backend).


**Persistent Volume (persistent):**
```yaml
volumes:
- name: data
  persistentVolumeClaim:
    claimName: pvc-storage  # Survives Pod deletion
```

## High-Level Understanding

**Why PV + PVC?**
- Separate storage concerns from Pod concerns
- Admin provisions storage
- Developer requests storage
- Kubernetes matches them

**Why it matters:**
- Stateful applications (databases) need persistent data
- Pods are ephemeral, storage is permanent
- Data survives Pod/Node failures (depending on backend)

**When to use:**
- Databases (PostgreSQL, MySQL, MongoDB)
- Caches (Redis, Memcached)
- File systems (shared logs, uploads)
- Anything where data must survive Pod death

**When not to use:**
- Stateless applications (web servers)
- Temporary data (caches, buffers)
- Read-only configuration

## Commands

```bash
# List PVs
kubectl get pv

# List PVCs
kubectl get pvc

# Describe PV
kubectl describe pv pv-name

# Describe PVC
kubectl describe pvc pvc-name

# Delete PVC (may delete underlying data based on reclaim policy)
kubectl delete pvc pvc-name

# Watch binding
kubectl get pvc -w
```

## Real-World Scenario

You deploy PostgreSQL to Kubernetes:
1. Storage Class auto-creates 20GB PV
2. PVC requests 20GB → binds to PV
3. Pod mounts PVC at `/var/lib/postgresql/data`
4. Database writes data to PVC
5. Pod crashes → Kubernetes restarts it
6. Pod mounts same PVC → Data still there
7. Node dies → Cloud storage keeps data safe
8. New node spins up → Pod mounts PVC again → Data accessible

Database is now truly persistent in Kubernetes.

## Realization

Pods die. Storage persists. That's the whole point.

---

**Progress: 47/90 days complete**


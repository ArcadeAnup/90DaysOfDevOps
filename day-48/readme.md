# Day 48: Persistent Volumes Hands-On (PV + PVC + Deployment)

## The Hands-On Workflow

Create PV → Create PVC → Create Deployment → Verify data persists


```bash
bash deploy-and-test.sh
```


## Verify with kubectl commands

```bash
# See all resources
kubectl get pv,pvc,deployments,pods -o wide

# Output:
# NAME                                 CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM
# persistentvolume/pv-local-storage    5Gi        RWO            Retain           Bound    default/pvc-local-storage
#
# NAME                                   STATUS   VOLUME               CAPACITY   ACCESS MODES
# persistentvolumeclaim/pvc-local-storage Bound   pv-local-storage     5Gi        RWO
#
# NAME                            READY   UP-TO-DATE   AVAILABLE
# deployment.apps/app-with-storage 1/1     1            1
#
# NAME                                        READY   STATUS    RESTARTS   AGE
# pod/app-with-storage-7d9f8f7f9f-xyz99      1/1     Running   0          2m
```

All connected and working.

## Describe to See Details

```bash
# Detailed PV info
kubectl describe pv pv-local-storage
# Shows: Capacity, Access Modes, Reclaim Policy, Status, Claim

# Detailed PVC info
kubectl describe pvc pvc-local-storage
# Shows: Status, Volume (which PV it's bound to), Capacity, Access Mode

# Detailed Deployment info
kubectl describe deployment app-with-storage
# Shows: Replicas, Pods, Volumes (references the PVC)
```

## Cleanup

```bash
# Delete Deployment
kubectl delete deployment app-with-storage

# Delete PVC
kubectl delete pvc pvc-local-storage

# Delete PV (Reclaim Policy: Retain means data stays on disk)
kubectl delete pv pv-local-storage

# Verify all deleted
kubectl get pv,pvc,deployments,pods
# Should be empty
```

## What We Verified

✅ PV created successfully
✅ PVC bound to PV
✅ Deployment mounted PVC
✅ Data written to volume
✅ Pod deleted (Deployment recreated it)
✅ New Pod accessed same data
✅ Data persisted across Pod death

## Key Realization from Day 48

**Theory (Day 47):** Understanding PV/PVC concept
**Practice (Day 48):** Actually seeing data persist



---

**Progress: 48/90 days complete**


# Task 08 — Volumes & Persistent Storage

## 🎯 Goal
Create a PersistentVolume (PV), claim it with a PersistentVolumeClaim (PVC), attach it to a Pod, write data to it, delete the Pod, recreate it — and prove the data survived.

---

## 📁 File
| File | Description |
|------|-------------|
| `volumes.yaml` | PV (1Gi hostPath), PVC (500Mi claim), and a Pod that writes timestamped data to the volume |

---

## 📖 YAML Breakdown

### PersistentVolume (PV)
```yaml
spec:
  storageClassName: manual
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /tmp/k8s-lab-data
  persistentVolumeReclaimPolicy: Retain
```
- **PV** is a cluster-level resource — not namespaced.
- **storageClassName** – How PV and PVC find each other (must match).
- **accessModes:**
  - `ReadWriteOnce` (RWO) – One node can read+write.
  - `ReadOnlyMany` (ROX) – Many nodes can read.
  - `ReadWriteMany` (RWX) – Many nodes can read+write (requires NFS/EFS).
- **hostPath** – Stores data in a directory on the Node. Fine for local dev, not for production.
- **Retain** – Data is preserved even after the PVC is deleted.

### PersistentVolumeClaim (PVC)
```yaml
spec:
  storageClassName: manual
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```
- PVC is **namespaced** — it lives in `k8s-lab`.
- Kubernetes finds the best matching PV (same `storageClassName`, access mode, and at least the requested capacity).

### Pod using the PVC
```yaml
volumeMounts:
  - name: persistent-storage
    mountPath: /data

volumes:
  - name: persistent-storage
    persistentVolumeClaim:
      claimName: lab-pvc
```
- The Pod references the PVC by name; the PVC resolves to the actual PV.

---

## 🚀 Steps

### 1. Apply everything
```bash
kubectl apply -f volumes.yaml
```

### 2. Check PV and PVC status
```bash
kubectl get pv lab-pv
kubectl get pvc lab-pvc -n k8s-lab
```
> PVC should show `STATUS: Bound` and be linked to `lab-pv`.

### 3. Verify data is being written
```bash
# Wait a moment, then read the log file
kubectl exec storage-demo -n k8s-lab -- cat /data/log.txt
```

### 4. 🔥 Delete the Pod — data should survive
```bash
kubectl delete pod storage-demo -n k8s-lab
```

### 5. Recreate the Pod
```bash
kubectl apply -f volumes.yaml
```

### 6. Verify old data is still there
```bash
kubectl exec storage-demo -n k8s-lab -- cat /data/log.txt
```
> You should see entries from BEFORE the Pod was deleted — proving persistence!

### 7. Inspect volume details
```bash
kubectl describe pv lab-pv
kubectl describe pvc lab-pvc -n k8s-lab
```

### 8. Clean up
```bash
kubectl delete -f volumes.yaml
# PV stays because of Retain policy — manually release if needed:
kubectl delete pv lab-pv
```

---

## 💡 Key Concepts

### The Storage Chain

```
Pod
 └── volumeMount (/data)
      └── volume (PVC reference)
           └── PersistentVolumeClaim (lab-pvc)
                └── PersistentVolume (lab-pv)
                     └── Actual Storage (hostPath / AWS EBS / NFS...)
```

### Access Modes

| Mode | Short | Description |
|------|-------|-------------|
| ReadWriteOnce | RWO | 1 node, read+write |
| ReadOnlyMany | ROX | Many nodes, read-only |
| ReadWriteMany | RWX | Many nodes, read+write |

### Reclaim Policies

| Policy | What happens when PVC is deleted |
|--------|----------------------------------|
| **Retain** | PV stays, data preserved, requires manual cleanup |
| **Delete** | PV and underlying storage auto-deleted (cloud) |
| **Recycle** | Data wiped, PV reused (deprecated) |

### emptyDir (ephemeral alternative)
```yaml
volumes:
  - name: temp
    emptyDir: {}   # Created when Pod starts, deleted when Pod stops
```
- Useful for temporary scratch space or sharing data between containers in a Pod.
- Data does NOT persist across Pod restarts.

---

## ✅ Expected Output

```
persistentvolume/lab-pv created
persistentvolumeclaim/lab-pvc created
pod/storage-demo created

NAME      STATUS   VOLUME   CAPACITY   ACCESS MODES
lab-pvc   Bound    lab-pv   1Gi        RWO

# After deleting and recreating Pod:
Pod started at: Mon Jan 13 10:00:00 UTC 2025   ← old entry
Heartbeat: Mon Jan 13 10:00:10 UTC 2025
Pod started at: Mon Jan 13 10:02:00 UTC 2025   ← new entry after restart
```

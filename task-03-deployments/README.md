# Task 03 — Deployments & ReplicaSets

## 🎯 Goal
Create a Deployment that manages multiple Pod replicas. Scale the Deployment up and down, and observe how Kubernetes self-heals by automatically replacing deleted Pods.

---

## 📁 File
| File | Description |
|------|-------------|
| `nginx-deployment.yaml` | A 3-replica nginx Deployment with rolling update strategy |

---

## 📖 YAML Breakdown

```yaml
apiVersion: apps/v1
kind: Deployment
```
- Deployments live in the **apps** API group (not the core `v1` group).

```yaml
spec:
  replicas: 3
```
- Kubernetes will ensure exactly **3 Pods** are running at all times.

```yaml
  selector:
    matchLabels:
      app: nginx
```
- The Deployment uses this selector to find **which Pods it owns**. It manages all Pods with `app: nginx`.

```yaml
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
```
- **RollingUpdate** – Replaces Pods gradually instead of all at once.
- **maxSurge: 1** – At most 1 extra Pod can exist above the desired count during an update.
- **maxUnavailable: 1** – At most 1 Pod can be unavailable during an update.

```yaml
  template:
    metadata:
      labels:
        app: nginx
```
- The **Pod template** – this is what gets stamped out for each replica. Labels here must match the selector above.

> ⚠️ The `selector.matchLabels` must match `template.metadata.labels`. If they don't, the Deployment will fail to manage its Pods.

---

## 🚀 Steps

### 1. Create the Deployment
```bash
kubectl apply -f nginx-deployment.yaml
```

### 2. Watch Pods come up
```bash
kubectl get pods -n k8s-lab -l app=nginx -w
```

### 3. Inspect the Deployment and ReplicaSet
```bash
# View the Deployment
kubectl get deployment nginx-deployment -n k8s-lab

# View the auto-created ReplicaSet
kubectl get replicaset -n k8s-lab

# Detailed view
kubectl describe deployment nginx-deployment -n k8s-lab
```

### 4. 🔥 Test self-healing — delete a Pod manually
```bash
# Get a Pod name
kubectl get pods -n k8s-lab -l app=nginx

# Delete one
kubectl delete pod <pod-name> -n k8s-lab

# Watch Kubernetes recreate it immediately
kubectl get pods -n k8s-lab -l app=nginx -w
```

### 5. Scale the Deployment
```bash
# Scale up to 5 replicas
kubectl scale deployment nginx-deployment --replicas=5 -n k8s-lab

# Scale down to 2
kubectl scale deployment nginx-deployment --replicas=2 -n k8s-lab

# Verify
kubectl get pods -n k8s-lab -l app=nginx
```

### 6. Clean up
```bash
kubectl delete -f nginx-deployment.yaml
```

---

## 💡 Key Concepts

| Concept | Description |
|---------|-------------|
| **Deployment** | Manages ReplicaSets and provides declarative updates |
| **ReplicaSet** | Ensures a specified number of identical Pods are running |
| **Self-healing** | K8s automatically replaces failed/deleted Pods |
| **RollingUpdate** | Zero-downtime deployment strategy |
| **selector** | The glue connecting a Deployment to its Pods |

---

## 🔄 Object Hierarchy

```
Deployment
  └── ReplicaSet  (manages versions)
        └── Pod  × replicas
              └── Container(s)
```

---

## ✅ Expected Output

```
deployment.apps/nginx-deployment created

NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           30s

# After deleting a Pod manually:
NAME                                READY   STATUS              AGE
nginx-deployment-abc123-xxxxx       1/1     Terminating         2m
nginx-deployment-abc123-yyyyy       0/1     ContainerCreating   1s   ← auto-recreated!
```

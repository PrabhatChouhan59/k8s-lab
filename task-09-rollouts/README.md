# Task 09 — Rollouts & Updates

## 🎯 Goal
Perform a zero-downtime rolling update on a Deployment, inspect rollout history, and roll back to a previous version when something goes wrong.

---

## 📁 File
| File | Description |
|------|-------------|
| `rollout.yaml` | A 4-replica nginx Deployment (v1.24) + Service, ready for update and rollback demos |

---

## 📖 YAML Breakdown

### Rolling Update Strategy
```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 1
```
- **maxSurge: 1** – At most 1 extra Pod above `replicas: 4` during update (total: 5 Pods max).
- **maxUnavailable: 1** – At most 1 Pod can be offline at a time.
- Result: The service stays up during the entire update — at least 3 Pods always ready.

### Change-cause Annotation
```yaml
annotations:
  kubernetes.io/change-cause: "Initial deployment — nginx v1.24"
```
- Recorded in rollout history when `--record` flag is used (or annotation is set).
- Lets you document what changed in each revision.

---

## 🚀 Steps

### 1. Deploy v1 (nginx 1.24)
```bash
kubectl apply -f rollout.yaml
kubectl rollout status deployment/rollout-demo -n k8s-lab
kubectl get pods -n k8s-lab -l app=rollout-demo
```

### 2. Update to v2 — change the image
```bash
kubectl set image deployment/rollout-demo \
  nginx=nginx:1.25-alpine \
  -n k8s-lab \
  --record
```

### 3. Watch the rolling update in real time
```bash
kubectl rollout status deployment/rollout-demo -n k8s-lab

# In a second terminal, watch Pods being replaced:
kubectl get pods -n k8s-lab -l app=rollout-demo -w
```
> You should see old Pods terminating while new Pods come up gradually.

### 4. Update to v3 — update with kubectl edit
```bash
kubectl edit deployment/rollout-demo -n k8s-lab
# Change: image: nginx:1.25-alpine → nginx:1.26-alpine
# Also update the change-cause annotation
# Save and exit
```

### 5. Or update by patching directly
```bash
kubectl patch deployment rollout-demo -n k8s-lab \
  -p '{"spec":{"template":{"spec":{"containers":[{"name":"nginx","image":"nginx:1.26-alpine"}]}}}}'
```

### 6. View rollout history
```bash
kubectl rollout history deployment/rollout-demo -n k8s-lab

# View details of a specific revision
kubectl rollout history deployment/rollout-demo -n k8s-lab --revision=1
kubectl rollout history deployment/rollout-demo -n k8s-lab --revision=2
```

### 7. 🔥 Simulate a bad deployment
```bash
# Deploy a broken image (wrong tag)
kubectl set image deployment/rollout-demo nginx=nginx:broken-tag -n k8s-lab

# Watch it fail
kubectl rollout status deployment/rollout-demo -n k8s-lab
kubectl get pods -n k8s-lab -l app=rollout-demo
# Pods will be in ImagePullBackOff / ErrImagePull
```

### 8. 🔙 Rollback to the previous version
```bash
# Rollback to previous revision
kubectl rollout undo deployment/rollout-demo -n k8s-lab

# Or rollback to a specific revision
kubectl rollout undo deployment/rollout-demo --to-revision=2 -n k8s-lab

# Verify recovery
kubectl rollout status deployment/rollout-demo -n k8s-lab
kubectl get pods -n k8s-lab -l app=rollout-demo
```

### 9. Pause and resume a rollout
```bash
# Pause mid-rollout (for canary-like testing)
kubectl rollout pause deployment/rollout-demo -n k8s-lab

# Make multiple changes while paused...
kubectl set image deployment/rollout-demo nginx=nginx:1.25-alpine -n k8s-lab

# Resume — all changes apply at once
kubectl rollout resume deployment/rollout-demo -n k8s-lab
```

### 10. Clean up
```bash
kubectl delete -f rollout.yaml
```

---

## 💡 Key Concepts

| Command | What it does |
|---------|-------------|
| `kubectl rollout status` | Watch rollout progress |
| `kubectl rollout history` | See revision history |
| `kubectl rollout undo` | Rollback to previous (or specific) revision |
| `kubectl rollout pause` | Freeze update mid-way |
| `kubectl rollout resume` | Continue a paused rollout |
| `kubectl set image` | Trigger update by changing container image |

### Rollout Strategies Comparison

| Strategy | Description | Downtime? |
|----------|-------------|-----------|
| **RollingUpdate** | Replace Pods gradually | ❌ None |
| **Recreate** | Kill all, then create all | ✅ Yes |

> Kubernetes also supports **Blue/Green** and **Canary** deployments using multiple Deployments + traffic splitting (via Ingress or a service mesh like Istio).

---

## ✅ Expected Output

```
# Rollout history
REVISION  CHANGE-CAUSE
1         Initial deployment — nginx v1.24
2         nginx:1.25-alpine
3         nginx:1.26-alpine

# During update:
NAME                           READY   STATUS              AGE
rollout-demo-abc-xxxx          1/1     Terminating         5m
rollout-demo-def-yyyy          1/1     Running             2s   ← new
rollout-demo-def-zzzz          0/1     ContainerCreating   0s   ← new

# After rollback:
deployment.apps/rollout-demo rolled back
```

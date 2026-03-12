# Task 05 — Labels, Selectors & Annotations

## 🎯 Goal
Organize Kubernetes resources using labels and selectors, and store non-identifying metadata using annotations. Learn to filter and target resources precisely using label queries.

---

## 📁 File
| File | Description |
|------|-------------|
| `labels-selectors.yaml` | Three Pods with different labels (env, tier, team) and a Service that selects only production frontend Pods |

---

## 📖 YAML Breakdown

### Labels
```yaml
labels:
  app: webapp
  env: production
  tier: frontend
  version: "2.0"
  team: platform
```
- **Labels** are key-value pairs attached to objects.
- Used by **selectors** to group and filter resources.
- Conventions: `app`, `env`, `tier`, `version`, `team` are common label keys.

### Annotations
```yaml
annotations:
  contact: "platform-team@company.com"
  docs: "https://wiki.company.com/webapp"
  description: "Production frontend — handle with care"
```
- **Annotations** store arbitrary metadata — NOT used for selection.
- Great for tooling, documentation, audit info, CI/CD metadata.

### Label Selector on Service
```yaml
spec:
  selector:
    app: webapp
    env: production
    tier: frontend
```
- Routes traffic **only** to Pods matching ALL three labels.
- The `app-staging` and `app-backend` Pods are excluded from this Service.

---

## 🚀 Steps

### 1. Apply all resources
```bash
kubectl apply -f labels-selectors.yaml
```

### 2. View Pods with their labels
```bash
kubectl get pods -n k8s-lab --show-labels
```

### 3. Filter by a single label
```bash
# All webapp Pods
kubectl get pods -n k8s-lab -l app=webapp

# Only production Pods
kubectl get pods -n k8s-lab -l env=production

# Only frontend Pods
kubectl get pods -n k8s-lab -l tier=frontend
```

### 4. Use set-based selectors
```bash
# Pods in production OR staging
kubectl get pods -n k8s-lab -l 'env in (production, staging)'

# Pods NOT in backend tier
kubectl get pods -n k8s-lab -l 'tier notin (backend)'

# Pods that have a 'team' label (any value)
kubectl get pods -n k8s-lab -l team
```

### 5. Multi-label selector (AND logic)
```bash
# Only production frontend Pods
kubectl get pods -n k8s-lab -l env=production,tier=frontend
```

### 6. View annotations
```bash
kubectl describe pod app-prod -n k8s-lab | grep -A 10 Annotations
```

### 7. Add a label to a running Pod
```bash
kubectl label pod app-staging env=uat -n k8s-lab --overwrite
kubectl get pods -n k8s-lab --show-labels
```

### 8. Remove a label
```bash
kubectl label pod app-staging env- -n k8s-lab
```

### 9. Check which Pods the Service targets
```bash
kubectl get endpoints frontend-prod-svc -n k8s-lab
```

### 10. Clean up
```bash
kubectl delete -f labels-selectors.yaml
```

---

## 💡 Key Concepts

| Feature | Purpose | Syntax |
|---------|---------|--------|
| **Label** | Identify/group resources | `key: value` |
| **Annotation** | Store metadata (not for selection) | `key: value` (any string) |
| **Equality selector** | Match exact label value | `-l env=production` |
| **Set-based selector** | Match a set of values | `-l 'env in (prod,staging)'` |
| **Negative selector** | Exclude values | `-l 'tier notin (backend)'` |

### Labels vs Annotations

| | Labels | Annotations |
|---|--------|-------------|
| **Used for selection?** | ✅ Yes | ❌ No |
| **Max length** | 63 chars | No limit |
| **Typical use** | Grouping, routing | Docs, tooling, audit |

---

## ✅ Expected Output

```
NAME          READY   STATUS    LABELS
app-prod      1/1     Running   app=webapp,env=production,tier=frontend,version=2.0
app-staging   1/1     Running   app=webapp,env=staging,tier=frontend,version=2.1-beta
app-backend   1/1     Running   app=webapp,env=production,tier=backend,version=1.5

# Endpoints for frontend-prod-svc (only app-prod matches all 3 selectors)
NAME                ENDPOINTS
frontend-prod-svc   10.244.0.x:80
```

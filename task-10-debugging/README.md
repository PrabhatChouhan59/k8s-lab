# Task 10 — Logs & Debugging

## 🎯 Goal
Master Kubernetes debugging using `kubectl logs`, `kubectl exec`, `kubectl describe`, and `kubectl get events`. Diagnose and understand the most common Pod failure states: `ImagePullBackOff`, `CrashLoopBackOff`, and `Pending`.

---

## 📁 File
| File | Description |
|------|-------------|
| `debugging.yaml` | One healthy Pod, three intentionally broken Pods (bad image, crash loop, pending), and a debug toolbox Pod |

---

## 📖 YAML Breakdown — Failure States

### ❌ ImagePullBackOff
```yaml
image: nginx:this-tag-does-not-exist
```
- Kubernetes can't pull the image from the registry.
- Common causes: wrong tag, private registry with no credentials, typo.

### ❌ CrashLoopBackOff
```yaml
command: ["sh", "-c", "echo 'Starting...'; exit 1"]
```
- Container starts but exits immediately (non-zero exit code).
- Kubernetes retries with exponential backoff (10s → 20s → 40s → max 5min).
- Common causes: bad entrypoint, missing config, application crash on startup.

### ❌ Pending
```yaml
resources:
  requests:
    cpu: "99"
    memory: "200Gi"
```
- No Node in the cluster has enough resources to satisfy the request.
- Pod stays in `Pending` indefinitely until resources free up.
- Other causes: taints, NodeSelector mismatch, PVC not bound.

### 🛠 Debug Toolbox Pod
```yaml
image: nicolaka/netshoot
command: ["sleep", "3600"]
```
- `netshoot` includes: `curl`, `dig`, `nslookup`, `ping`, `nmap`, `tcpdump`, `wget`, `ip`, and more.
- Keeps running for 1 hour so you can `exec` into it to debug networking.

---

## 🚀 Steps

### 1. Apply all Pods
```bash
kubectl apply -f debugging.yaml
```

### 2. List all Pods and observe their states
```bash
kubectl get pods -n k8s-lab -l task=10-debugging
```

### 3. 🔍 Debug: View logs
```bash
# Healthy Pod logs
kubectl logs debug-healthy -n k8s-lab

# Crash loop — logs from current failed run
kubectl logs debug-crashloop -n k8s-lab

# Crash loop — logs from the PREVIOUS failed run
kubectl logs debug-crashloop -n k8s-lab --previous

# Follow logs in real time
kubectl logs debug-healthy -n k8s-lab -f

# Last 50 lines only
kubectl logs debug-healthy -n k8s-lab --tail=50

# Logs with timestamps
kubectl logs debug-healthy -n k8s-lab --timestamps
```

### 4. 🔍 Debug: describe (the most powerful command)
```bash
# For ImagePullBackOff — look at Events section
kubectl describe pod debug-bad-image -n k8s-lab

# For CrashLoopBackOff — look at Last State and Events
kubectl describe pod debug-crashloop -n k8s-lab

# For Pending — look at Events (e.g. "Insufficient cpu")
kubectl describe pod debug-pending -n k8s-lab
```
> The **Events** section at the bottom of `describe` is always your first stop.

### 5. 🔍 Debug: exec into a running container
```bash
# Open interactive shell
kubectl exec -it debug-healthy -n k8s-lab -- sh

# Inside: check processes, env, filesystem
ps aux
env
ls /etc/nginx
cat /etc/nginx/nginx.conf
exit
```

### 6. 🔍 Debug: Use the network toolbox Pod
```bash
kubectl exec -it debug-toolbox -n k8s-lab -- sh

# Inside the toolbox:
# Test DNS resolution
nslookup kubernetes.default.svc.cluster.local
dig nginx-clusterip.k8s-lab.svc.cluster.local

# Test connectivity to another service
curl -v http://nginx-clusterip.k8s-lab.svc.cluster.local

# Check network interfaces
ip addr show
exit
```

### 7. 🔍 Debug: Cluster events
```bash
# All events in namespace
kubectl get events -n k8s-lab

# Sort by time (most recent last)
kubectl get events -n k8s-lab --sort-by='.lastTimestamp'

# Only warning events
kubectl get events -n k8s-lab --field-selector type=Warning
```

### 8. 🔍 Debug: Resource usage
```bash
# CPU and memory per Node
kubectl top nodes

# CPU and memory per Pod
kubectl top pods -n k8s-lab
```
> `kubectl top` requires the **Metrics Server** to be installed.

### 9. Fix the bad image Pod
```bash
# Patch the bad Pod with a correct image
kubectl delete pod debug-bad-image -n k8s-lab

# Edit the YAML, fix the image tag, and re-apply
kubectl apply -f debugging.yaml
```

### 10. Clean up
```bash
kubectl delete -f debugging.yaml
```

---

## 💡 Debugging Cheat Sheet

| Symptom | First Command | Common Cause |
|---------|--------------|-------------|
| `ImagePullBackOff` | `kubectl describe pod` → Events | Wrong image tag, missing registry credentials |
| `CrashLoopBackOff` | `kubectl logs --previous` | Bad entrypoint, missing config, app crash |
| `Pending` | `kubectl describe pod` → Events | Not enough resources, no matching Node, PVC not bound |
| `OOMKilled` | `kubectl describe pod` → Last State | Memory limit too low |
| `Running` but broken | `kubectl exec` → inspect inside | Config errors, wrong env vars |
| Network issues | `kubectl exec` toolbox → `curl`/`dig` | DNS, firewall, wrong service name |

### The Golden Debugging Loop

```
kubectl get pods          → Spot the problem Pod
  ↓
kubectl describe pod      → Read Events section
  ↓
kubectl logs (--previous) → Read error messages
  ↓
kubectl exec -it          → Inspect from inside
  ↓
kubectl get events        → Cluster-wide view
```

---

## ✅ Expected Pod States

```
NAME                READY   STATUS             RESTARTS
debug-healthy       1/1     Running            0
debug-bad-image     0/1     ImagePullBackOff   0
debug-crashloop     0/1     CrashLoopBackOff   3
debug-pending       0/1     Pending            0
debug-toolbox       1/1     Running            0
```

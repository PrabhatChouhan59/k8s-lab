# Task 02 — Pods

## 🎯 Goal
Create a Pod using a YAML manifest, access it, view its logs, and delete it. Understand the full lifecycle of the most fundamental Kubernetes object.

---

## 📁 File
| File | Description |
|------|-------------|
| `nginx-pod.yaml` | Runs an nginx web server inside a Pod with health probes and resource limits |

---

## 📖 YAML Breakdown

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  namespace: k8s-lab
  labels:
    app: nginx
  annotations:
    description: "A basic nginx Pod for learning purposes"
```
- **labels** – Key-value tags used to select/filter resources.
- **annotations** – Non-identifying metadata (notes, tool hints, etc.).

```yaml
spec:
  containers:
    - name: nginx
      image: nginx:1.25-alpine
      ports:
        - containerPort: 80
```
- **containers** – One or more containers running inside the Pod. They share the same network and storage.
- **containerPort** – Documents which port the container listens on (does not expose it externally on its own).

```yaml
      resources:
        requests:
          memory: "64Mi"
          cpu: "100m"
        limits:
          memory: "128Mi"
          cpu: "200m"
```
- **requests** – Minimum resources the container needs; used for scheduling decisions.
- **limits** – Maximum resources the container can use. If exceeded, it gets throttled (CPU) or killed (memory).
- **100m** = 100 millicores = 0.1 CPU core.

```yaml
      readinessProbe:
        httpGet:
          path: /
          port: 80
        initialDelaySeconds: 5
        periodSeconds: 5
```
- **readinessProbe** – Kubernetes only sends traffic to the Pod once this probe passes.

```yaml
      livenessProbe:
        httpGet:
          path: /
          port: 80
        initialDelaySeconds: 10
        periodSeconds: 10
```
- **livenessProbe** – If this fails, Kubernetes restarts the container automatically.

---

## 🚀 Steps

### 1. Apply the Pod
```bash
kubectl apply -f nginx-pod.yaml
```

### 2. Watch it start up
```bash
kubectl get pod nginx-pod -n k8s-lab -w
```
(`-w` = watch for changes)

### 3. Describe the Pod (events, status, probes)
```bash
kubectl describe pod nginx-pod -n k8s-lab
```

### 4. View logs
```bash
kubectl logs nginx-pod -n k8s-lab

# Follow logs in real time
kubectl logs nginx-pod -n k8s-lab -f
```

### 5. Execute a command inside the container
```bash
kubectl exec -it nginx-pod -n k8s-lab -- sh

# Inside the shell:
wget -qO- localhost     # check nginx response
exit
```

### 6. Port-forward to access nginx locally
```bash
kubectl port-forward pod/nginx-pod 8080:80 -n k8s-lab
# Open http://localhost:8080 in your browser
```

### 7. Delete the Pod
```bash
kubectl delete pod nginx-pod -n k8s-lab
# or
kubectl delete -f nginx-pod.yaml
```

---

## 💡 Key Concepts

| Concept | Description |
|---------|-------------|
| **Pod** | One or more containers sharing network + storage |
| **containerPort** | Documents the port; does not expose it externally |
| **Resource requests/limits** | CPU/memory guardrails for scheduling and stability |
| **readinessProbe** | Controls when traffic is sent to the Pod |
| **livenessProbe** | Controls when the container is restarted |
| **restartPolicy** | `Always` / `OnFailure` / `Never` |

---

## ✅ Expected Output

```
pod/nginx-pod created

NAME        READY   STATUS    RESTARTS   AGE
nginx-pod   1/1     Running   0          15s

<!DOCTYPE html>
<html>
<head><title>Welcome to nginx!</title>...
```

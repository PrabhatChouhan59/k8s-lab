# Task 01 — Kubernetes Setup & CLI Basics

## 🎯 Goal
Install Kind and kubectl, spin up a local cluster, and verify everything is working using the Kubernetes CLI.

---

## 📁 File
| File | Description |
|------|-------------|
| `cluster-info.yaml` | Creates a `k8s-lab` Namespace and a test Pod to confirm the cluster is healthy |

---

## 📖 YAML Breakdown

### Namespace
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: k8s-lab
```
- **apiVersion: v1** – Core Kubernetes API group.
- **kind: Namespace** – Logical isolation boundary within the cluster.
- **name: k8s-lab** – All tasks in this lab will use this namespace.

### Pod (Verification)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hello-k8s
  namespace: k8s-lab
spec:
  containers:
    - name: hello
      image: busybox:1.36
      command: ["sh", "-c", "echo '✅ Kubernetes is working!' && sleep 30"]
  restartPolicy: Never
```
- **image: busybox** – Lightweight Linux image, perfect for quick tests.
- **command** – Runs a shell command, prints a success message, then sleeps so you can inspect it.
- **restartPolicy: Never** – The Pod completes once and does not restart.

---

## 🚀 Steps

### 1. Apply the YAML
```bash
kubectl apply -f cluster-info.yaml
```

### 2. Check the Namespace was created
```bash
kubectl get namespace k8s-lab
```

### 3. Watch the Pod come up
```bash
kubectl get pod hello-k8s -n k8s-lab
```

### 4. View the Pod logs (your success message)
```bash
kubectl logs hello-k8s -n k8s-lab
```

### 5. Explore core kubectl commands
```bash
# View all nodes in the cluster
kubectl get nodes

# Detailed info about the cluster
kubectl cluster-info

# Current context (which cluster you're talking to)
kubectl config current-context

# List all contexts
kubectl config get-contexts

# Describe a node
kubectl describe node <node-name>

# List all resources across all namespaces
kubectl get all --all-namespaces
```

### 6. Clean up
```bash
kubectl delete -f cluster-info.yaml
```

---

## 💡 Key Concepts

| Concept | Description |
|---------|-------------|
| **Node** | A machine (VM or physical) in the cluster |
| **Namespace** | Virtual partition inside a cluster |
| **Pod** | Smallest deployable unit; wraps one or more containers |
| **kubectl** | CLI tool to interact with Kubernetes API |
| **kubeconfig** | File (`~/.kube/config`) that stores cluster connection details |

---

## ✅ Expected Output

```
namespace/k8s-lab created
pod/hello-k8s created

NAME        STATUS      RESTARTS   AGE
hello-k8s   Completed   0          10s

✅ Kubernetes is working!
```

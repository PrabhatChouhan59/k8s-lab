# ☸️ k8s-lab — Kubernetes Hands-On Lab

A structured, beginner-to-advanced Kubernetes learning lab with **10 progressive tasks**. Each task contains a production-quality YAML manifest and a detailed README that explains every line of code.

> No cloud account needed. Runs 100% locally with **Kind** + **kubectl**.

---

## 🎯 Who Is This For?

- Developers learning Kubernetes for the first time
- DevOps engineers brushing up on core K8s concepts
- Anyone preparing for the **CKA / CKAD** certification exams

---

## 🗂️ Repository Structure

```
k8s-lab/
│
├── 📄 README.md                          ← You are here
├── 📄 how-to-install-kind-kubectl.md     ← Start here! Setup guide
│
├── 📁 task-01-setup/
│   ├── cluster-info.yaml                 ← Namespace + hello-world Pod
│   └── README.md                         ← Setup & kubectl basics
│
├── 📁 task-02-pods/
│   ├── nginx-pod.yaml                    ← Pod with probes & resource limits
│   └── README.md                         ← Pod lifecycle, logs, exec
│
├── 📁 task-03-deployments/
│   ├── nginx-deployment.yaml             ← 3-replica Deployment
│   └── README.md                         ← Scaling, self-healing, ReplicaSets
│
├── 📁 task-04-services/
│   ├── nginx-service.yaml                ← ClusterIP + NodePort services
│   └── README.md                         ← Service types, DNS, port-forward
│
├── 📁 task-05-labels/
│   ├── labels-selectors.yaml             ← Multi-label Pods + Service selector
│   └── README.md                         ← Labels, annotations, selectors
│
├── 📁 task-06-configmaps/
│   ├── configmap.yaml                    ← ConfigMap with env vars + files
│   └── README.md                         ← 3 ways to inject config into Pods
│
├── 📁 task-07-secrets/
│   ├── secret.yaml                       ← Opaque + TLS secrets
│   └── README.md                         ← Secrets, base64, security best practices
│
├── 📁 task-08-volumes/
│   ├── volumes.yaml                      ← PV + PVC + Pod with persistent storage
│   └── README.md                         ← Storage chain, access modes, reclaim policy
│
├── 📁 task-09-rollouts/
│   ├── rollout.yaml                      ← Deployment with rolling update + Service
│   └── README.md                         ← Updates, history, rollback, pause/resume
│
└── 📁 task-10-debugging/
    ├── debugging.yaml                    ← Healthy + 3 broken Pods + toolbox
    └── README.md                         ← Logs, exec, describe, events, top
```

---

## 🗺️ Learning Path

```
[Setup]         Task 01 → Install Kind & kubectl, verify cluster
[Compute]       Task 02 → Pods (the building block of everything)
[Compute]       Task 03 → Deployments & ReplicaSets (scaling + self-healing)
[Networking]    Task 04 → Services (ClusterIP, NodePort, DNS)
[Organization]  Task 05 → Labels, Selectors & Annotations
[Configuration] Task 06 → ConfigMaps (inject config without rebuilding images)
[Configuration] Task 07 → Secrets (secure credentials management)
[Storage]       Task 08 → Volumes & Persistent Storage (PV, PVC)
[Operations]    Task 09 → Rolling Updates & Rollbacks
[Operations]    Task 10 → Logs & Debugging (troubleshooting toolkit)
```

---

## ⚡ Quick Start

### Step 1 — Install tools
Follow the [installation guide](./how-to-install-kind-kubectl.md) to install Docker, Kind, and kubectl.

### Step 2 — Create a cluster
```bash
kind create cluster --name k8s-lab
kubectl get nodes
```

### Step 3 — Create the lab namespace
```bash
kubectl create namespace k8s-lab
```

### Step 4 — Start with Task 01
```bash
cd task-01-setup
kubectl apply -f cluster-info.yaml
kubectl get pods -n k8s-lab
```

### Step 5 — Work through each task
Each task folder has a `README.md` — read it, apply the YAML, and run every command.

---

## 🧰 Useful kubectl Commands

```bash
# General
kubectl get all -n k8s-lab          # See everything in the namespace
kubectl get events -n k8s-lab       # See cluster events
kubectl api-resources               # List all resource types

# Pods
kubectl get pods -n k8s-lab -w      # Watch pod status changes
kubectl describe pod <name> -n k8s-lab
kubectl logs <pod> -n k8s-lab -f    # Follow logs
kubectl exec -it <pod> -n k8s-lab -- sh

# Deployments
kubectl rollout status deployment/<name> -n k8s-lab
kubectl rollout history deployment/<name> -n k8s-lab
kubectl rollout undo deployment/<name> -n k8s-lab

# Cleanup
kubectl delete namespace k8s-lab    # Deletes ALL resources in the namespace
```

---

## 📚 Topics Covered

| # | Task | Core Concepts |
|---|------|--------------|
| 01 | Setup & CLI Basics | Nodes, Namespaces, kubectl, kubeconfig |
| 02 | Pods | Container spec, probes, resource limits, restartPolicy |
| 03 | Deployments & ReplicaSets | Replicas, self-healing, scaling, selector |
| 04 | Services | ClusterIP, NodePort, DNS, endpoints |
| 05 | Labels & Annotations | Label selectors, set-based queries, metadata |
| 06 | ConfigMaps | env, envFrom, volume mounts, live updates |
| 07 | Secrets | base64, Opaque, TLS, security practices |
| 08 | Volumes & Storage | PV, PVC, accessModes, reclaimPolicy, hostPath |
| 09 | Rollouts & Updates | RollingUpdate, history, undo, pause/resume |
| 10 | Logs & Debugging | logs, exec, describe, events, top, netshoot |

---

## 🧹 Full Cleanup

When you're done with the entire lab:

```bash
# Delete all lab resources
kubectl delete namespace k8s-lab

# Delete the Kind cluster
kind delete cluster --name k8s-lab
```

---

## 🔗 Further Learning

- [Kubernetes Official Docs](https://kubernetes.io/docs/)
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [Kind Documentation](https://kind.sigs.k8s.io/)
- [CKAD Curriculum](https://github.com/cncf/curriculum)
- [Killer.sh CKAD Practice](https://killer.sh/)

---

## 📝 License

MIT — free to use, fork, share, and learn from.

---

<p align="center">
  Made with ❤️ for the Kubernetes learning community
</p>

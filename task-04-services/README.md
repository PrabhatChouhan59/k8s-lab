# Task 04 — Services

## 🎯 Goal
Expose the nginx Deployment using two types of Services: **ClusterIP** (internal cluster communication) and **NodePort** (external access). Understand how Kubernetes networking and service discovery work.

---

## 📋 Prerequisite
Task 03's Deployment (`nginx-deployment`) must be running.
```bash
kubectl apply -f ../task-03-deployments/nginx-deployment.yaml
```

---

## 📁 File
| File | Description |
|------|-------------|
| `nginx-service.yaml` | Defines a ClusterIP and a NodePort service for nginx |

---

## 📖 YAML Breakdown

### ClusterIP Service
```yaml
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
```
- **ClusterIP** – Default service type. Only reachable from within the cluster.
- **selector: app: nginx** – Routes traffic to any Pod with this label.
- **port** – The port the Service listens on (inside the cluster).
- **targetPort** – The port on the actual Pod containers.

### NodePort Service
```yaml
spec:
  type: NodePort
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
```
- **NodePort** – Opens a static port on every Node in the cluster.
- **nodePort: 30080** – Any traffic hitting `<NodeIP>:30080` is forwarded to the Pods.
- Valid range: **30000–32767**.

---

## 🚀 Steps

### 1. Apply services
```bash
kubectl apply -f nginx-service.yaml
```

### 2. List services
```bash
kubectl get svc -n k8s-lab
```

### 3. Describe a service
```bash
kubectl describe svc nginx-clusterip -n k8s-lab
kubectl describe svc nginx-nodeport -n k8s-lab
```

### 4. Test ClusterIP (from inside the cluster)
```bash
# Run a temporary Pod and curl the ClusterIP service by DNS name
kubectl run test-pod --image=busybox:1.36 --rm -it --restart=Never -n k8s-lab -- \
  wget -qO- http://nginx-clusterip.k8s-lab.svc.cluster.local
```

### 5. Access via NodePort

**With Kind:**
```bash
# Get the internal node IP
kubectl get nodes -o wide

# Port-forward as an alternative (easier with Kind)
kubectl port-forward svc/nginx-nodeport 8080:80 -n k8s-lab
# Open http://localhost:8080
```

**With Minikube:**
```bash
minikube service nginx-nodeport -n k8s-lab
```

### 6. Inspect Endpoints (Pods behind the Service)
```bash
kubectl get endpoints nginx-clusterip -n k8s-lab
```

### 7. Clean up
```bash
kubectl delete -f nginx-service.yaml
```

---

## 💡 Key Concepts

| Service Type | Accessible From | Use Case |
|--------------|----------------|----------|
| **ClusterIP** | Inside cluster only | Microservice-to-microservice communication |
| **NodePort** | Outside cluster via `NodeIP:NodePort` | Dev/testing external access |
| **LoadBalancer** | Public internet (cloud) | Production external access |
| **ExternalName** | DNS alias | Redirecting to external DNS |

### DNS Pattern Inside Cluster
```
<service-name>.<namespace>.svc.cluster.local
nginx-clusterip.k8s-lab.svc.cluster.local
```

---

## ✅ Expected Output

```
service/nginx-clusterip created
service/nginx-nodeport created

NAME               TYPE        CLUSTER-IP      PORT(S)        AGE
nginx-clusterip    ClusterIP   10.96.100.50    80/TCP         10s
nginx-nodeport     NodePort    10.96.200.75    80:30080/TCP   10s
```

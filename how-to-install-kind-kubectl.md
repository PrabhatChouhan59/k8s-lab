# 📦 How to Install Kind & kubectl

This guide walks you through installing **Kind** (Kubernetes in Docker) and **kubectl** on Linux, macOS, and Windows — so you can run a local Kubernetes cluster on your laptop.

---

## 📋 Prerequisites

- **Docker Desktop** or **Docker Engine** must be installed and running.
  - [Install Docker Desktop](https://www.docker.com/products/docker-desktop/)
  - Verify: `docker version`

---

## 1️⃣ Install kubectl

kubectl is the Kubernetes CLI — it talks to any Kubernetes cluster.

### 🐧 Linux

```bash
# Download the latest stable release
curl -LO "https://dl.k8s.io/release/$(curl -Ls https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

# Make it executable
chmod +x kubectl

# Move to PATH
sudo mv kubectl /usr/local/bin/

# Verify
kubectl version --client
```

### 🍎 macOS (Homebrew — recommended)

```bash
brew install kubectl

# Verify
kubectl version --client
```

### 🍎 macOS (manual)

```bash
# Apple Silicon (M1/M2/M3)
curl -LO "https://dl.k8s.io/release/$(curl -Ls https://dl.k8s.io/release/stable.txt)/bin/darwin/arm64/kubectl"

# Intel Mac
curl -LO "https://dl.k8s.io/release/$(curl -Ls https://dl.k8s.io/release/stable.txt)/bin/darwin/amd64/kubectl"

chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client
```

### 🪟 Windows (Chocolatey)

```powershell
choco install kubernetes-cli
kubectl version --client
```

### 🪟 Windows (manual)

```powershell
# In PowerShell as Administrator:
$version = (Invoke-WebRequest "https://dl.k8s.io/release/stable.txt").Content.Trim()
Invoke-WebRequest "https://dl.k8s.io/release/$version/bin/windows/amd64/kubectl.exe" -OutFile kubectl.exe

# Move to a directory in your PATH (e.g., C:\Windows\System32)
Move-Item kubectl.exe C:\Windows\System32\

kubectl version --client
```

---

## 2️⃣ Install Kind

Kind (Kubernetes in Docker) runs local Kubernetes clusters using Docker containers as nodes.

### 🐧 Linux

```bash
# AMD64 / x86_64
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.23.0/kind-linux-amd64

# ARM64 (e.g. Raspberry Pi, ARM server)
# curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.23.0/kind-linux-arm64

chmod +x kind
sudo mv kind /usr/local/bin/

# Verify
kind version
```

### 🍎 macOS (Homebrew — recommended)

```bash
brew install kind

# Verify
kind version
```

### 🍎 macOS (manual)

```bash
# Apple Silicon
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.23.0/kind-darwin-arm64

# Intel Mac
# curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.23.0/kind-darwin-amd64

chmod +x kind
sudo mv kind /usr/local/bin/
kind version
```

### 🪟 Windows (Chocolatey)

```powershell
choco install kind
kind version
```

### 🪟 Windows (manual)

```powershell
curl.exe -Lo kind-windows-amd64.exe https://kind.sigs.k8s.io/dl/v0.23.0/kind-windows-amd64
Move-Item kind-windows-amd64.exe C:\Windows\System32\kind.exe
kind version
```

---

## 3️⃣ Create Your First Kind Cluster

```bash
# Create a single-node cluster named "k8s-lab"
kind create cluster --name k8s-lab

# Verify the cluster is running
kubectl cluster-info --context kind-k8s-lab
kubectl get nodes
```

### Multi-node cluster (optional)

Save the following as `kind-config.yaml`:

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: k8s-lab
nodes:
  - role: control-plane
  - role: worker
  - role: worker
```

Then create it:

```bash
kind create cluster --config kind-config.yaml
kubectl get nodes   # Should show 3 nodes
```

---

## 4️⃣ Manage Clusters

```bash
# List all Kind clusters
kind get clusters

# Switch kubectl context to a Kind cluster
kubectl config use-context kind-k8s-lab

# List all kubectl contexts
kubectl config get-contexts

# Delete a cluster
kind delete cluster --name k8s-lab
```

---

## 5️⃣ Optional Tools

These tools are not required for the lab but are highly recommended for a better experience:

### kubectx + kubens — fast context/namespace switching

```bash
# macOS
brew install kubectx

# Linux
sudo git clone https://github.com/ahmetb/kubectx /opt/kubectx
sudo ln -s /opt/kubectx/kubectx /usr/local/bin/kubectx
sudo ln -s /opt/kubectx/kubens /usr/local/bin/kubens
```

```bash
kubectx kind-k8s-lab   # Switch cluster
kubens k8s-lab         # Switch default namespace
```

### k9s — terminal UI for Kubernetes

```bash
# macOS
brew install k9s

# Linux
curl -sS https://webi.sh/k9s | sh
```

```bash
k9s   # Launch the TUI
```

### kubectl aliases (add to your ~/.bashrc or ~/.zshrc)

```bash
alias k='kubectl'
alias kgp='kubectl get pods'
alias kgs='kubectl get svc'
alias kgd='kubectl get deployment'
alias kdp='kubectl describe pod'
alias kl='kubectl logs'
alias kex='kubectl exec -it'
alias kns='kubectl config set-context --current --namespace'
```

---

## ✅ Verification Checklist

Run these commands — all should succeed:

```bash
docker version           # Docker is running
kubectl version --client # kubectl is installed
kind version             # Kind is installed
kind get clusters        # Shows: k8s-lab
kubectl get nodes        # Shows: k8s-lab-control-plane   Ready
kubectl get namespaces   # Shows: default, kube-system, etc.
```

---

## 🆚 Kind vs Minikube

| Feature | Kind | Minikube |
|---------|------|---------|
| **Speed** | Fast to start | Slightly slower |
| **Multi-node** | ✅ Easy | Limited |
| **CI/CD friendly** | ✅ Yes | Less so |
| **Resource usage** | Lower | Higher |
| **LoadBalancer support** | Needs MetalLB | Built-in tunnel |
| **Best for** | Advanced labs, CI | Beginners |

Both work for this lab. Kind is preferred.

---

> 💡 **Tip:** If you ever break your cluster, just delete it and recreate it in seconds — that's the beauty of Kind!
>
> ```bash
> kind delete cluster --name k8s-lab
> kind create cluster --name k8s-lab
> ```

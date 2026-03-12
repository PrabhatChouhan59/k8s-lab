# Task 06 — ConfigMaps

## 🎯 Goal
Decouple application configuration from container images using ConfigMaps. Inject config data into Pods as environment variables and as mounted config files — three different ways.

---

## 📁 File
| File | Description |
|------|-------------|
| `configmap.yaml` | A ConfigMap with env vars + config files, and a Pod that consumes them all three ways |

---

## 📖 YAML Breakdown

### The ConfigMap
```yaml
data:
  APP_ENV: "production"      # Simple key-value
  LOG_LEVEL: "info"

  app.properties: |          # Multi-line file stored as a key
    server.port=8080
    ...
```
- ConfigMap data is plain text only (not encrypted).
- Keys can be simple strings or entire file contents using `|` (block scalar).

### Method 1 — Specific keys as env vars
```yaml
env:
  - name: APP_ENV
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: APP_ENV
```
- Picks a single key from the ConfigMap and injects it as a named env var.
- Useful when you want control over which vars are exposed.

### Method 2 — All keys as env vars
```yaml
envFrom:
  - configMapRef:
      name: app-config
```
- Dumps ALL keys from the ConfigMap as environment variables.
- Fast and convenient, but less explicit.

### Method 3 — Mounted as files
```yaml
volumeMounts:
  - name: config-volume
    mountPath: /etc/app-config

volumes:
  - name: config-volume
    configMap:
      name: app-config
      items:
        - key: app.properties
          path: app.properties
```
- ConfigMap keys become **files** inside the container.
- Files are updated automatically when the ConfigMap changes (with a short delay).

---

## 🚀 Steps

### 1. Apply everything
```bash
kubectl apply -f configmap.yaml
```

### 2. Inspect the ConfigMap
```bash
kubectl get configmap app-config -n k8s-lab
kubectl describe configmap app-config -n k8s-lab
```

### 3. Wait for Pod to be ready
```bash
kubectl get pod configmap-demo -n k8s-lab -w
```

### 4. Verify environment variables inside the Pod
```bash
kubectl exec -it configmap-demo -n k8s-lab -- sh

# Inside the container:
env | grep -E 'APP|LOG|MAX|PORT'    # See injected env vars
exit
```

### 5. Verify mounted config files
```bash
kubectl exec -it configmap-demo -n k8s-lab -- sh

# Inside the container:
ls /etc/app-config/
cat /etc/app-config/app.properties

ls /etc/nginx/conf.d/
cat /etc/nginx/conf.d/default.conf
exit
```

### 6. Update a ConfigMap value and observe
```bash
kubectl edit configmap app-config -n k8s-lab
# Change LOG_LEVEL from "info" to "debug", save and exit.

# Mounted files update within ~60 seconds:
kubectl exec -it configmap-demo -n k8s-lab -- cat /etc/app-config/app.properties
# Note: env vars do NOT update; Pod must be restarted for env var changes.
```

### 7. Create a ConfigMap from files using CLI
```bash
echo "PORT=9090" > myconfig.env
kubectl create configmap cli-config --from-env-file=myconfig.env -n k8s-lab
kubectl get configmap cli-config -n k8s-lab -o yaml
```

### 8. Clean up
```bash
kubectl delete -f configmap.yaml
```

---

## 💡 Key Concepts

| Method | How | Live Update? |
|--------|-----|-------------|
| `env.valueFrom.configMapKeyRef` | Single key → env var | ❌ Need Pod restart |
| `envFrom.configMapRef` | All keys → env vars | ❌ Need Pod restart |
| `volumes + volumeMounts` | Keys → files | ✅ Auto-updated (~60s) |

> ⚠️ ConfigMaps are **not encrypted**. Never store passwords or tokens in a ConfigMap — use Secrets (Task 07) instead.

---

## ✅ Expected Output

```bash
# env vars inside Pod
APP_ENV=production
LOG_LEVEL=info
MAX_CONNECTIONS=100
APP_PORT=8080

# /etc/app-config/app.properties
server.port=8080
server.host=0.0.0.0
logging.level=info
db.pool.size=10
feature.dark_mode=true
```

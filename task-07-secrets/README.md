# Task 07 — Secrets

## 🎯 Goal
Store sensitive data (passwords, API keys, TLS certificates) in Kubernetes Secrets and inject them into Pods securely — without hardcoding credentials in your application code or Docker images.

---

## 📁 File
| File | Description |
|------|-------------|
| `secret.yaml` | An Opaque Secret and TLS Secret, with a Pod consuming them via env vars and volume mounts |

---

## ⚠️ Important Security Note
> Kubernetes Secrets are **base64-encoded**, NOT encrypted by default. Base64 is encoding, not encryption — anyone with cluster access can decode them.
>
> For production, consider:
> - **Sealed Secrets** (Bitnami)
> - **External Secrets Operator** with AWS Secrets Manager / HashiCorp Vault
> - **Kubernetes encryption at rest** (encrypts etcd)

---

## 📖 YAML Breakdown

### Encoding Secret values
```bash
# Encode a value
echo -n 'myS3cr3tP@ss' | base64
# Output: bXlTM2NyM3RQQHNz

# Decode a value
echo 'bXlTM2NyM3RQQHNz' | base64 -d
# Output: myS3cr3tP@ss
```

> Always use `-n` with `echo` to avoid encoding a trailing newline.

### Secret types
```yaml
type: Opaque                  # Generic secrets (most common)
type: kubernetes.io/tls       # TLS certificates
type: kubernetes.io/dockerconfigjson  # Docker registry credentials
type: kubernetes.io/basic-auth        # Username + password
```

### Method 1 — Env vars
```yaml
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: app-secret
        key: db-password
```
- Decoded automatically — the container sees the plain text value.

### Method 2 — Volume mount (preferred for sensitive data)
```yaml
volumeMounts:
  - name: secret-volume
    mountPath: /etc/secrets
    readOnly: true

volumes:
  - name: secret-volume
    secret:
      secretName: app-secret
      defaultMode: 0400   # chmod: owner read-only
```
- Each key becomes a **file** under `/etc/secrets/`.
- More secure than env vars (not exposed in `ps`, process inspection, or logs).

---

## 🚀 Steps

### 1. Create Secrets from CLI (preferred in practice)
```bash
# From literal values (no YAML needed)
kubectl create secret generic app-secret \
  --from-literal=db-password='myS3cr3tP@ss' \
  --from-literal=api-key='abcd-1234-efgh' \
  --from-literal=username='admin' \
  -n k8s-lab

# From a file
kubectl create secret generic tls-secret \
  --from-file=tls.crt=./cert.crt \
  --from-file=tls.key=./cert.key \
  -n k8s-lab
```

### 2. Or apply from YAML
```bash
kubectl apply -f secret.yaml
```

### 3. List Secrets (values are hidden)
```bash
kubectl get secrets -n k8s-lab
```

### 4. Describe a Secret (shows metadata, NOT values)
```bash
kubectl describe secret app-secret -n k8s-lab
```

### 5. Decode a Secret value
```bash
kubectl get secret app-secret -n k8s-lab -o jsonpath='{.data.db-password}' | base64 -d
```

### 6. Wait for Pod and check env vars
```bash
kubectl get pod secret-demo -n k8s-lab -w

kubectl exec -it secret-demo -n k8s-lab -- sh
env | grep -E 'DB_|API_'   # See decoded values in env
exit
```

### 7. Check mounted secret files
```bash
kubectl exec -it secret-demo -n k8s-lab -- sh
ls -la /etc/secrets/
cat /etc/secrets/db-password   # Plain text value
cat /etc/secrets/api-key
exit
```

### 8. Clean up
```bash
kubectl delete -f secret.yaml
```

---

## 💡 Key Concepts

| | Secrets | ConfigMaps |
|---|---------|-----------|
| **Data type** | Sensitive (passwords, keys) | Non-sensitive (config) |
| **Storage** | base64 encoded in etcd | Plain text in etcd |
| **Encryption** | Not by default | No |
| **Use for** | Credentials, certs, tokens | App config, settings |

### Env vars vs Volume mounts

| | Env Vars | Volume Mount |
|---|---------|-------------|
| **Live update** | ❌ Restart needed | ✅ Auto-updated |
| **Visibility** | Exposed in env dump | Restricted by file perms |
| **Best for** | Simple values | Files, certs |

---

## ✅ Expected Output

```bash
# Inside the container
DB_PASSWORD=myS3cr3tP@ss
API_KEY=abcd-1234-efgh
DB_USER=admin

# /etc/secrets/
-r-------- db-password
-r-------- api-key
-r-------- username
```

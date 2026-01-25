# Level 3: ConfigMaps and Secrets

## 🎯 Learning Objectives
- Understand why configuration should be separated from code
- Learn the difference between ConfigMaps and Secrets
- Practice injecting configuration into containers

## 📚 Concepts Covered

### The Problem
In Levels 1 and 2, we hardcoded configuration values directly in the Deployment manifests:
```yaml
env:
  - name: DB_PASSWORD
    value: "password"  # 😱 Password visible in plain text!
```

This is problematic because:
1. **Security risk**: Secrets are visible in version control
2. **Inflexibility**: Changing config requires redeploying
3. **Code duplication**: Same values repeated across deployments

### The Solution: ConfigMaps and Secrets

| Resource | Purpose | Stored As | Use For |
|----------|---------|-----------|---------|
| **ConfigMap** | Non-sensitive config | Plain text | DB host, app settings, feature flags |
| **Secret** | Sensitive data | Base64 encoded | Passwords, API keys, certificates |

> ⚠️ **Important**: Secrets are only base64 encoded, NOT encrypted! For production, use:
> - Sealed Secrets
> - External Secrets Operator
> - Vault integration

### Ways to Use ConfigMaps/Secrets

**1. Environment variables (individual keys):**
```yaml
env:
  - name: DB_HOST
    valueFrom:
      configMapKeyRef:
        name: kubecoin-config
        key: DB_HOST
```

**2. Environment from entire ConfigMap/Secret:**
```yaml
envFrom:
  - configMapRef:
      name: kubecoin-config
  - secretRef:
      name: kubecoin-secrets
```

**3. Volume mounts (for files):**
```yaml
volumes:
  - name: config-volume
    configMap:
      name: kubecoin-config
```

## ⚠️ What's Different from Level 2

| Aspect | Level 2 | Level 3 |
|--------|---------|---------|
| DB credentials | Hardcoded in YAML | Stored in Secret |
| App config | Hardcoded in YAML | Stored in ConfigMap |
| Changing config | Redeploy required | Update ConfigMap/Secret |
| Security | ❌ Visible in git | ✅ Separated from code |

## 🚀 Deployment Steps

```bash
# 1. Apply ConfigMap and Secret FIRST
kubectl apply -f k8s/level-3-configmaps-secrets/configmap.yaml
kubectl apply -f k8s/level-3-configmaps-secrets/secret.yaml

# 2. Verify they exist
kubectl get configmap kubecoin-config -o yaml
kubectl get secret kubecoin-secrets -o yaml

# 3. Apply remaining resources
kubectl apply -f k8s/level-3-configmaps-secrets/

# 4. Verify environment variables are injected
kubectl exec -it $(kubectl get pod -l app=backend -o jsonpath='{.items[0].metadata.name}') -- env | grep DB_
```

## 📁 Files in This Level

| File | Purpose |
|------|---------|
| `configmap.yaml` | **NEW** - Non-sensitive configuration |
| `secret.yaml` | **NEW** - Sensitive credentials (base64 encoded) |
| `postgres-pvc.yaml` | Same as Level 2 |
| `postgres-deployment.yaml` | **MODIFIED** - Uses envFrom |
| `postgres-service.yaml` | Same as Level 2 |
| `backend-deployment.yaml` | **MODIFIED** - Uses envFrom |
| `backend-service.yaml` | Same as Level 2 |
| `frontend-deployment.yaml` | **MODIFIED** - Uses configMapRef |
| `frontend-service.yaml` | Same as Level 2 |

## 🔐 Creating Secrets

### Method 1: YAML with base64 encoding
```bash
# Encode values
echo -n 'postgres' | base64    # cG9zdGdyZXM=
echo -n 'password' | base64    # cGFzc3dvcmQ=
```

### Method 2: kubectl create (recommended)
```bash
kubectl create secret generic kubecoin-secrets \
  --from-literal=POSTGRES_USER=postgres \
  --from-literal=POSTGRES_PASSWORD=password \
  --from-literal=DB_USER=postgres \
  --from-literal=DB_PASSWORD=password \
  --dry-run=client -o yaml > secret.yaml
```

## 🔍 Verification Commands

```bash
# View ConfigMap
kubectl describe configmap kubecoin-config

# View Secret (values are hidden)
kubectl describe secret kubecoin-secrets

# Decode a secret value
kubectl get secret kubecoin-secrets -o jsonpath='{.data.POSTGRES_PASSWORD}' | base64 -d

# Verify env vars in pod
kubectl exec -it $(kubectl get pod -l app=backend -o jsonpath='{.items[0].metadata.name}') -- printenv | grep -E 'DB_|POSTGRES_'
```

## 💡 Discussion Questions

1. Why are Secrets base64 encoded instead of encrypted?
2. What happens if you update a ConfigMap? Does the pod automatically restart?
3. How would you rotate database credentials in production?
4. What's the difference between `env` with `valueFrom` vs `envFrom`?

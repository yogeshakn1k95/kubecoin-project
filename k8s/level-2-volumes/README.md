# Level 2: Kubernetes with Persistent Volumes & StatefulSets

## 🎯 Learning Objectives
- Understand StatefulSets vs Deployments
- Learn about Headless Services
- Understand PersistentVolumes (PV) and PersistentVolumeClaims (PVC)
- See how stateful applications are managed in Kubernetes

## 📚 Concepts Covered

### The Problem with Deployments for Databases
In Level 1, PostgreSQL uses a Deployment which:
- Creates pods with random names (postgres-abc123)
- Data is lost when pod restarts
- No stable network identity

### The Solution: StatefulSets

**StatefulSet** provides:
- **Stable pod names**: postgres-0, postgres-1, etc.
- **Stable network identity**: Each pod gets a DNS name
- **Ordered deployment**: Pods are created/deleted in order
- **Persistent storage**: Data survives pod restarts

### Headless Service

A **Headless Service** (clusterIP: None) provides:
- Direct DNS resolution to pod IPs
- No load balancing (you connect directly to specific pods)
- Required for StatefulSet network identity

**DNS Format**: `<pod-name>.<headless-service>.<namespace>.svc.cluster.local`
- Example: `postgres-0.postgres-headless.default.svc.cluster.local`

### PersistentVolume (PV) vs PersistentVolumeClaim (PVC)

| Resource | Description |
|----------|-------------|
| **PV** | The actual storage (provisioned by admin) |
| **PVC** | A request/claim for storage (used by pods) |

## ⚠️ What's Different from Level 1

| Aspect | Level 1 | Level 2 |
|--------|---------|---------|
| PostgreSQL | Deployment | **StatefulSet** |
| Service type | ClusterIP | **Headless (clusterIP: None)** |
| Pod names | Random (postgres-abc123) | **Stable (postgres-0)** |
| Storage | None | **PV + PVC** |
| Data persistence | ❌ Lost on restart | ✅ Persisted |

## 🚀 Deployment Steps

```bash
# 1. Apply all manifests
kubectl apply -f k8s/level-2-volumes/

# 2. Check StatefulSet status
kubectl get statefulset
kubectl get pods -l app=postgres

# 3. Check PV and PVC status
kubectl get pv
kubectl get pvc

# 4. Wait for postgres pod to be ready
kubectl wait --for=condition=ready pod -l app=postgres --timeout=60s

# 5. Initialize the database (run the init script manually)
kubectl exec -it postgres-0 -- psql -U postgres -d kubecoin -c "
CREATE TABLE IF NOT EXISTS wallets (
    id VARCHAR(255) PRIMARY KEY,
    balance DECIMAL(15, 2) DEFAULT 1000.00,
    coins DECIMAL(15, 2) DEFAULT 0.00,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
CREATE INDEX IF NOT EXISTS idx_wallets_id ON wallets(id);
"

# 6. Access the application
# For Minikube:
minikube service frontend-service
```

## 📁 Files in This Level

| File | Purpose |
|------|---------|
| `postgres-pv.yaml` | PersistentVolume definition |
| `postgres-pvc.yaml` | PersistentVolumeClaim |
| `postgres-headless-service.yaml` | Headless service for StatefulSet |
| `postgres-statefulset.yaml` | StatefulSet (replaces Deployment) |
| `backend-deployment.yaml` | Flask backend (updated DB_HOST) |
| `backend-service.yaml` | Backend ClusterIP service |
| `frontend-deployment.yaml` | React/Nginx frontend |
| `frontend-service.yaml` | Frontend NodePort service |

## 🔍 Verification Commands

```bash
# Check StatefulSet
kubectl describe statefulset postgres

# Notice the stable pod name (postgres-0, not postgres-abc123)
kubectl get pods -l app=postgres -o wide

# Test headless service DNS
kubectl run -it --rm debug --image=busybox --restart=Never -- nslookup postgres-headless

# Connect to postgres using stable pod name
kubectl exec -it postgres-0 -- psql -U postgres -d kubecoin -c "\dt"

# Test data persistence - delete the pod and watch it come back
kubectl delete pod postgres-0
kubectl get pods -w  # Watch it recreate with same name postgres-0

# Verify data is still there after pod restart!
kubectl exec -it postgres-0 -- psql -U postgres -d kubecoin -c "SELECT * FROM wallets;"
```

## 💡 Discussion Questions

1. Why do databases need StatefulSets instead of Deployments?
2. What happens if you scale the StatefulSet to 2 replicas?
3. How is the headless service different from a regular ClusterIP service?
4. What is the pod deletion order in a StatefulSet?
5. After deleting postgres-0, why does the new pod still have the data?

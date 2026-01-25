# Level 1: Basic Kubernetes Manifests

## 🎯 Learning Objectives
- Understand Kubernetes Deployments and how they manage Pods
- Learn about Services and how they enable network communication
- Deploy a multi-tier application (frontend, backend, database)

## 📚 Concepts Covered

### Deployments
A **Deployment** tells Kubernetes how to create and update instances of your application. Key fields:
- `replicas`: Number of pod instances to run
- `selector`: How the Deployment finds its Pods
- `template`: Pod specification (containers, ports, env vars)

### Services
A **Service** exposes your Pods to network traffic. Types used here:
- **ClusterIP** (default): Internal cluster access only
- **NodePort**: Exposes on each Node's IP at a static port

## ⚠️ Important Notes

**This level has intentional limitations for teaching:**
1. **No persistent storage** - PostgreSQL data is lost when the pod restarts!
2. **Hardcoded credentials** - Passwords are visible in plain text
3. **No resource limits** - Containers can consume unlimited resources
4. **Manual database setup** - You need to run the init script manually

These will be addressed in subsequent levels.

## 🚀 Deployment Steps

```bash
# 1. Apply all manifests
kubectl apply -f k8s/level-1-basic/

# 2. Check deployment status
kubectl get pods
kubectl get services

# 3. Wait for postgres pod to be ready
kubectl wait --for=condition=ready pod -l app=postgres --timeout=60s

# 4. Initialize the database (run the init script manually)
kubectl exec -it $(kubectl get pod -l app=postgres -o jsonpath='{.items[0].metadata.name}') -- psql -U postgres -d kubecoin -c "
CREATE TABLE IF NOT EXISTS wallets (
    id VARCHAR(255) PRIMARY KEY,
    balance DECIMAL(15, 2) DEFAULT 1000.00,
    coins DECIMAL(15, 2) DEFAULT 0.00,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
CREATE INDEX IF NOT EXISTS idx_wallets_id ON wallets(id);
"

# 5. Access the application
# Frontend is available at http://<any-node-ip>:30000
```

## 📁 Files in This Level

| File | Purpose |
|------|---------|
| `postgres-deployment.yaml` | PostgreSQL database deployment |
| `postgres-service.yaml` | Internal service for database access |
| `backend-deployment.yaml` | Flask API deployment |
| `backend-service.yaml` | Internal service for backend access |
| `frontend-deployment.yaml` | React/Nginx frontend deployment |
| `frontend-service.yaml` | External NodePort service |

## 🔍 Verification Commands

```bash
# Check all resources
kubectl get all

# View pod logs
kubectl logs -l app=backend
kubectl logs -l app=postgres

# Test backend health
kubectl exec -it $(kubectl get pod -l app=frontend -o jsonpath='{.items[0].metadata.name}') -- curl http://backend-service:5000/health

# Connect to postgres and check tables
kubectl exec -it $(kubectl get pod -l app=postgres -o jsonpath='{.items[0].metadata.name}') -- psql -U postgres -d kubecoin -c "\dt"

# Watch pod status
kubectl get pods -w
```

## 💡 Discussion Questions

1. What happens if you delete the PostgreSQL pod? (`kubectl delete pod -l app=postgres`)
2. Why can't you access `backend-service` from outside the cluster?
3. What's the difference between a Deployment and a Pod?
4. Why did we have to run the init script manually?

# Expensy - Expense Management Platform

## Overview
Expensy is a full-stack expense management application running on **Azure Kubernetes Service (AKS)** in West Europe. Infrastructure is managed with **Terraform** and automated deployments via **GitHub Actions**.

## Your Infrastructure

### Azure Setup
- **Resource Group**: `project3-rg` (West Europe)
- **AKS Cluster**: `project3-aks-cluster`
- **Terraform State**: Stored in `expensytfstate789` storage account (tfstate-rg)
- **DNS Prefix**: `expensy`
- **Environment Tag**: Production

### Kubernetes Cluster
- **Node Pool**: `default`
- **Node Count**: 2 nodes
- **Node Size**: Standard_B2s_v2 (2 vCPU, 4GB RAM per node)
- **Identity**: System Assigned (Azure Managed Identity)

### Applications Deployed
- **Frontend**: Next.js on port 3000
- **Backend**: Node.js API on port 8706
- **Database**: MongoDB (10GB PVC)
- **Cache**: Redis (10GB PVC)
- **Monitoring**: Prometheus & Grafana (10GB PVC)

### Domain & Access
- **Domain**: https://expensy-lara.westeurope.cloudapp.azure.com
- **SSL/TLS**: Let's Encrypt (cert-manager)
- **Ingress**: NGINX with automatic HTTPS redirect

## Quick Start

### Local Development (Docker Compose)

```bash
# Start all services locally
docker-compose up -d

# Services will be available at:
# Frontend: http://localhost:3000
# Backend: http://localhost:8706
# MongoDB: localhost:27017 (credentials: root/example)
# Redis: localhost:6379 (password: someredispassword)

# Stop services
docker-compose down
```

### Deploy to AKS (Automated)

Push to GitHub and GitHub Actions automatically:
1. Builds backend & frontend
2. Runs tests
3. Deploys to AKS cluster `project3-aks-cluster`
4. Restarts pods in `expensy` namespace

**Trigger branches**: `main`, `cicd`, `monitoring`, `security`

### Manual Kubernetes Commands

```bash
# Get cluster credentials
az aks get-credentials --resource-group project3-rg --name project3-aks-cluster

# Check deployment status
kubectl get pods -n expensy
kubectl get svc -n expensy
kubectl get ingress -n expensy

# View backend logs
kubectl logs -f deployment/backend -n expensy

# Access MongoDB
kubectl exec -it deployment/mongodb -n expensy -- mongosh
```

## File Structure

```
PROJECT3/
├── devops.expensy/
│   ├── .github/workflows/
│   │   └── ci-cd.yaml              # Deploys to project3-aks-cluster
│   ├── k8s/                        # Kubernetes manifests
│   │   ├── namespace.yaml          # Creates 'expensy' namespace
│   │   ├── configmap.yaml          # Config for both services
│   │   ├── secrets.yaml            # MongoDB & Redis credentials
│   │   ├── backend-deployment.yaml # Connects to MongoDB & Redis
│   │   ├── frontend-deployment.yaml
│   │   ├── mongo-deployment.yaml   # 1GB PVC mount at /data/db
│   │   ├── redis-deployment.yaml   # 1GB PVC mount at /data
│   │   ├── ingress.yaml            # Routes to expensy domain
│   │   └── pvc-deployment.yaml     # Persistent volumes
│   ├── monitoring/                 # Prometheus & Grafana
│   ├── security/                   # cert-manager, network policies
│   └── docker-compose.yaml         # Local development
│
├── expensy_backend/
├── expensy_frontend/
│
└── terraform/
    └── main.tf                     # Provisions project3-rg & AKS cluster
```

## Environment Variables

### Backend (deployed to AKS)
```
PORT=8706
DATABASE_URI= *******************
REDIS_HOST=redis
REDIS_PORT=6379
REDIS_PASSWORD= ******************
NEXT_PUBLIC_API_URL=https://expensy-lara.westeurope.cloudapp.azure.com
```

### Frontend (deployed to AKS)
```
NEXT_PUBLIC_API_URL=https://expensy-lara.westeurope.cloudapp.azure.com
```

## Kubernetes Namespace & Resources

All application resources run in the **`expensy`** namespace:

```bash
# View all resources in expensy namespace
kubectl get all -n expensy

# View persistent volumes
kubectl get pvc -n expensy

# View configurations
kubectl get configmap -n expensy
kubectl get secrets -n expensy
```

### Pod Replicas
- **Backend**: 1 replica
- **Frontend**: 1 replica
- **MongoDB**: 1 replica
- **Redis**: 1 replica

Scale replicas in deployment files:
```yaml
spec:
  replicas: 2  # Change from 1 to 2
```

## Monitoring Stack

Deployed to Kubernetes:
- **Prometheus**: Collects metrics from pods
- **Grafana**: Visualization dashboards
- **Node Exporter**: System-level metrics from nodes

Access Grafana through port-forward:
```bash
kubectl port-forward -n expensy svc/grafana 3000:3000
# Open http://108.141.104.31:3000/
```

## Troubleshooting

### Check Pod Status
```bash
kubectl describe pod <pod-name> -n expensy
kubectl logs <pod-name> -n expensy
```

### MongoDB Connection Issues
```bash
# Test MongoDB connectivity from backend pod
kubectl exec -it deployment/backend -n expensy -- \
  mongosh mongodb://root:example@mongodb:27017
```

### Redis Connection Issues
```bash
# Test Redis from backend pod
kubectl exec -it deployment/backend -n expensy -- \
  redis-cli -h redis -a someredispassword ping
```

### AKS Cluster Issues
```bash
# View cluster info
kubectl cluster-info
kubectl get nodes
kubectl describe node <node-name>

# Check resource usage
kubectl top nodes
kubectl top pods -n expensy
```

### Ingress/DNS Not Working
```bash
# Check ingress status
kubectl get ingress -n expensy
kubectl describe ingress expensy-ingress -n expensy

# Check certificate status
kubectl get certificate -n expensy
kubectl describe certificate expensy-tls-secret -n expensy
```

## Updating Your Infrastructure

### Scale Nodes
Edit `terraform/main.tf`:
```hcl
default_node_pool {
  node_count = 3  # Changed from 2
}
```

Apply changes:
```bash
cd terraform
terraform plan
terraform apply
```

### Change Node Type
```hcl
vm_size = "Standard_B4ms"  # Changed from Standard_B2s_v2
```

### Add Tags
```hcl
tags = {
  Environment = "Prod"
  Team        = "DevOps"
  CostCenter  = "Engineering"
}
```

## CI/CD Workflow

1. Push code to `main`, `cicd`, `monitoring`, or `security` branch
2. GitHub Actions automatically:
   - Builds backend & frontend
   - Runs tests
   - Deploys to `project3-aks-cluster`
   - Applies k8s manifests
   - Restarts pods

View deployment status:
```bash
# In GitHub: Actions tab
# Or check AKS:
kubectl rollout status deployment/backend -n expensy
kubectl rollout status deployment/frontend -n expensy
```

## Documentation

- 📋 [Security Guide](./SECURITY.md) - Specific to your infrastructure
- 📋 [Compliance & Best Practices](./COMPLIANCE.md) - Your configuration details
- 📋 [This README](./README.md)
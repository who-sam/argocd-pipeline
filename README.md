# MIND - ArgoCD GitOps Pipeline

[![ArgoCD](https://img.shields.io/badge/ArgoCD-GitOps-EF7B4D.svg)](https://argoproj.github.io/cd/)
[![Kubernetes](https://img.shields.io/badge/Kubernetes-1.31-326CE5.svg)](https://kubernetes.io/)
[![GitOps](https://img.shields.io/badge/GitOps-Automated-00ADD8.svg)](https://www.gitops.tech/)

> Kubernetes manifests for GitOps-based continuous deployment with ArgoCD.

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Manifests Structure](#manifests-structure)
- [Prerequisites](#prerequisites)
- [Setup Instructions](#setup-instructions)
- [Deployment Resources](#deployment-resources)
- [GitOps Workflow](#gitops-workflow)
- [Operations](#operations)
- [Troubleshooting](#troubleshooting)

---

## 🎯 Overview

This repository contains Kubernetes manifests for deploying the MIND application using ArgoCD's GitOps methodology. Changes to manifests automatically trigger deployments to the EKS cluster.

### What Gets Deployed

✅ **Namespace** - Isolated environment for application  
✅ **Secrets** - Database credentials and JWT keys  
✅ **PostgreSQL** - StatefulSet with persistent storage  
✅ **Backend** - Go API deployment with 2 replicas  
✅ **Frontend** - React SPA deployment with 2 replicas  
✅ **Services** - LoadBalancer services for external access  

---

## 🏗️ Architecture

### Deployment Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        Kubernetes Cluster                       │
│                                                                 │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                    Namespace: notes-app                   │  │
│  │                                                           │  │
│  │  ┌─────────────────────────────────────────────────────┐  │  │
│  │  │         Frontend Service                            │  │  │
│  │  │         Type: LoadBalancer                          │  │  │
│  │  │         Port: 80                                    │  │  │
│  │  └────────┬────────────────────────────────────────────┘  │  │
│  │           │                                               │  │
│  │  ┌────────▼────────────────────────────────────────────┐  │  │
│  │  │         Frontend Deployment                         │  │  │
│  │  │         Replicas: 2                                 │  │  │
│  │  │         Image: whosam1/notes-app-frontend           │  │  │
│  │  │         Port: 80                                    │  │  │
│  │  └─────────────────────────────────────────────────────┘  │  │
│  │                                                           │  │
│  │  ┌─────────────────────────────────────────────────────┐  │  │
│  │  │         Backend Service                             │  │  │
│  │  │         Type: LoadBalancer                          │  │  │
│  │  │         Port: 8080                                  │  │  │
│  │  └────────┬────────────────────────────────────────────┘  │  │
│  │           │                                               │  │
│  │  ┌────────▼────────────────────────────────────────────┐  │  │
│  │  │         Backend Deployment                          │  │  │
│  │  │         Replicas: 2                                 │  │  │
│  │  │         Image: whosam1/notes-app-backend            │  │  │
│  │  │         Port: 8080                                  │  │  │
│  │  └────────┬────────────────────────────────────────────┘  │  │
│  │           │                                               │  │
│  │  ┌────────▼────────────────────────────────────────────┐  │  │
│  │  │         PostgreSQL Service                          │  │  │
│  │  │         Type: ClusterIP                             │  │  │
│  │  │         Port: 5432                                  │  │  │
│  │  └────────┬────────────────────────────────────────────┘  │  │
│  │           │                                               │  │
│  │  ┌────────▼────────────────────────────────────────────┐  │  │
│  │  │         PostgreSQL StatefulSet                      │  │  │
│  │  │         Replicas: 1                                 │  │  │
│  │  │         PVC: 10Gi gp3                               │  │  │
│  │  │         Port: 5432                                  │  │  │
│  │  └─────────────────────────────────────────────────────┘  │  │
│  │                                                           │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### GitOps Flow

```
┌──────────────────────────────────────────────────────────────┐
│                      Developer Workflow                      │
└───────────────┬──────────────────────────────────────────────┘
                │
                ▼
┌──────────────────────────────────────────────────────────────┐
│  CI Pipeline Updates Manifests (image tags)                  │
│  └─ Commit & push to mind-argocd-pipeline repo               │
└───────────────┬──────────────────────────────────────────────┘
                │
                ▼
┌──────────────────────────────────────────────────────────────┐
│  ArgoCD Detects Changes                                      │
│  ├─ Polls repository every 3 minutes                         │
│  ├─ Or: Webhook notification (instant)                       │
│  └─ Compares desired state (Git) vs actual state (Cluster)   │
└───────────────┬──────────────────────────────────────────────┘
                │
                ▼
┌──────────────────────────────────────────────────────────────┐
│  ArgoCD Syncs Application                                    │
│  ├─ Creates/updates Kubernetes resources                     │
│  ├─ Applies manifests in correct order                       │
│  ├─ Waits for health checks                                  │
│  └─ Reports sync status                                      │
└───────────────┬──────────────────────────────────────────────┘
                │
                ▼
┌──────────────────────────────────────────────────────────────┐
│  Application Running                                         │
│  ├─ Pods ready and serving traffic                           │
│  ├─ Services exposed via LoadBalancer                        │
│  └─ ArgoCD continuously monitors health                      │
└──────────────────────────────────────────────────────────────┘
```

---

## 📁 Manifests Structure

```
mind-argocd-pipeline/
├── namespace.yaml                  # Namespace definition
├── postgres-secrets.yaml           # Database credentials
├── postgres-pvc-simple.yaml        # Persistent Volume Claim
├── postgres-deployment.yaml        # Database StatefulSet
├── postgres-service.yaml           # Database internal service
├── backend-deployment.yaml         # Backend deployment
├── backend-service-simple.yaml     # Backend LoadBalancer
├── frontend-deployment.yaml        # Frontend deployment
├── frontend-service-simple.yaml    # Frontend LoadBalancer
├── sc-gp3-csi.yaml                # Storage class (gp3)
└── README.md                       # This file
```

### Manifest Dependencies

```
1. namespace.yaml
    └─ Creates: notes-app namespace

2. postgres-secrets.yaml
    └─ Creates: Secrets for DB credentials

3. sc-gp3-csi.yaml
    └─ Creates: StorageClass for EBS volumes

4. postgres-pvc-simple.yaml
    └─ Creates: PVC for PostgreSQL data

5. postgres-deployment.yaml
    └─ Creates: PostgreSQL StatefulSet
    └─ Depends on: PVC, Secrets

6. postgres-service.yaml
    └─ Creates: ClusterIP service for DB

7. backend-deployment.yaml
    └─ Creates: Backend pods
    └─ Depends on: postgres-service, Secrets

8. backend-service-simple.yaml
    └─ Creates: LoadBalancer for backend

9. frontend-deployment.yaml
    └─ Creates: Frontend pods

10. frontend-service-simple.yaml
    └─ Creates: LoadBalancer for frontend
```

---

## ✅ Prerequisites

### Required Tools

| Tool | Version | Purpose |
|------|---------|---------|
| **kubectl** | 1.28+ | Kubernetes CLI |
| **argocd CLI** | Latest | ArgoCD management |
| **AWS CLI** | 2.x | AWS operations |

### Cluster Requirements

- EKS cluster running Kubernetes 1.31
- ArgoCD installed on cluster
- EBS CSI driver installed (for persistent storage)
- Sufficient resources:
  - CPU: 2+ cores available
  - Memory: 4+ GB available
  - Storage: 10+ GB available

### Install ArgoCD

```bash
aws eks update-kubeconfig --name my-eks-project-dev-cluster --region us-east-1

# Create ArgoCD namespace
kubectl create namespace argocd

# Install ArgoCD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Wait for pods to be ready
kubectl wait --for=condition=Ready pods --all -n argocd --timeout=300s

# Get admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

# Port-forward to access UI
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Access ArgoCD UI: https://localhost:8080

---

## 🚀 Setup Instructions

### 1. Clone Repository

```bash
git clone https://github.com/who-sam/mind-argocd-pipeline.git
cd mind-argocd-pipeline
```

### 2. Configure Secrets

Edit `postgres-secrets.yaml`:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: postgres-secrets
  namespace: notes-app
type: Opaque
stringData:
  POSTGRES_DB: notes_app
  POSTGRES_USER: postgres
  POSTGRES_PASSWORD: "CHANGE_THIS_PASSWORD"  # Change in production
  JWT_SECRET: "CHANGE_THIS_SECRET"            # Change in production
```

**Important:** Use strong passwords in production!

### 3. Create ArgoCD Application

#### Method 1: Using kubectl

```bash
# Create application manifest
cat <<EOF | kubectl apply -f -
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: notes-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/who-sam/mind-argocd-pipeline.git
    targetRevision: main
    path: .
  destination:
    server: https://kubernetes.default.svc
    namespace: notes-app
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
EOF
```

#### Method 2: Using ArgoCD CLI

```bash
# Login to ArgoCD
argocd login localhost:8080

# Create application
argocd app create notes-app \
  --repo https://github.com/who-sam/mind-argocd-pipeline.git \
  --path . \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace notes-app \
  --sync-policy automated \
  --self-heal \
  --sync-option CreateNamespace=true
```

#### Method 3: Using ArgoCD UI

1. Click **+ NEW APP**
2. Configure:
   - **Application Name**: notes-app
   - **Project**: default
   - **Sync Policy**: Automatic
   - **Repository URL**: https://github.com/who-sam/mind-argocd-pipeline.git
   - **Revision**: main
   - **Path**: .
   - **Cluster URL**: https://kubernetes.default.svc
   - **Namespace**: notes-app
3. Click **CREATE**

### 4. Sync Application

```bash
# Trigger sync
argocd app sync notes-app

# Watch sync progress
argocd app get notes-app --watch
```

### 5. Verify Deployment

```bash
# Check all resources
kubectl get all -n notes-app

# Check pods
kubectl get pods -n notes-app

# Expected output:
# NAME                        READY   STATUS    RESTARTS   AGE
# backend-xxx-yyy             1/1     Running   0          2m
# backend-xxx-zzz             1/1     Running   0          2m
# frontend-xxx-yyy            1/1     Running   0          2m
# frontend-xxx-zzz            1/1     Running   0          2m
# postgres-0                  1/1     Running   0          3m
```

### 6. Get Application URL

```bash
# Get LoadBalancer URLs
kubectl get svc -n notes-app

# Expected output:
# NAME               TYPE           EXTERNAL-IP                    PORT(S)
# frontend-service   LoadBalancer   a123.us-east-1.elb.amazonaws.com   80:30080/TCP
# backend-service    LoadBalancer   b456.us-east-1.elb.amazonaws.com   8080:30808/TCP

# Access application
# Frontend: http://<frontend-external-ip>
# Backend API: http://<backend-external-ip>:8080/api/health
```

---

## 📦 Deployment Resources

### Namespace

**File:** `namespace.yaml`

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: notes-app
  labels:
    name: notes-app
    app: notes-application
```

### Secrets

**File:** `postgres-secrets.yaml`

Contains:
- Database credentials
- JWT secret
- Connection parameters

### PostgreSQL StatefulSet

**File:** `postgres-deployment.yaml`

- **Image**: postgres:15-alpine
- **Replicas**: 1
- **Storage**: 10Gi EBS volume
- **Resources**:
  - CPU: 250m - 500m
  - Memory: 256Mi - 512Mi

### Backend Deployment

**File:** `backend-deployment.yaml`

- **Image**: whosam1/notes-app-backend:latest
- **Replicas**: 2
- **Port**: 8080
- **Resources**:
  - CPU: 100m - 200m
  - Memory: 128Mi - 256Mi
- **Init Container**: Waits for PostgreSQL

### Frontend Deployment

**File:** `frontend-deployment.yaml`

- **Image**: whosam1/notes-app-frontend:latest
- **Replicas**: 2
- **Port**: 80
- **Resources**:
  - CPU: 50m - 100m
  - Memory: 64Mi - 128Mi

### Services

**LoadBalancer Services:**
- Frontend: Port 80
- Backend: Port 8080

**ClusterIP Service:**
- PostgreSQL: Port 5432 (internal only)

---

## 🔄 GitOps Workflow

### Automated Deployment Flow

```
1. Developer commits code to MIND repository
2. Jenkins CI pipeline builds Docker images
3. Jenkins updates image tags in this repository
4. ArgoCD detects manifest changes
5. ArgoCD syncs application to cluster
6. Kubernetes performs rolling update
7. New version deployed with zero downtime
```

### Manual Operations

#### Sync Application
```bash
argocd app sync notes-app
```

#### View Application Status
```bash
argocd app get notes-app
```

#### View Sync History
```bash
argocd app history notes-app
```

#### Rollback to Previous Version
```bash
# Get history
argocd app history notes-app

# Rollback to specific revision
argocd app rollback notes-app <revision-number>
```

### Sync Policies

Current configuration:
- **Auto-sync**: Enabled
- **Self-heal**: Enabled (auto-fixes manual changes)
- **Prune**: Enabled (removes deleted resources)

---

## 🔧 Operations

### Scale Deployments

#### Scale Backend
```bash
# Edit backend-deployment.yaml
replicas: 4  # Change from 2 to 4

# Commit and push
git add backend-deployment.yaml
git commit -m "Scale backend to 4 replicas"
git push

# ArgoCD auto-syncs
```

#### Scale Frontend
```bash
kubectl scale deployment frontend -n notes-app --replicas=4

# Or update manifest and push
```

### Update Application Version

#### CI Pipeline (Automatic)
```bash
# CI pipeline automatically updates image tags
# Example: whosam1/notes-app-backend:abc123d
```

#### Manual Update
```bash
# Edit deployment manifest
image: whosam1/notes-app-backend:new-version

# Commit and push
git commit -am "Update backend to new-version"
git push
```

### View Logs

```bash
# Frontend logs
kubectl logs -f deployment/frontend -n notes-app

# Backend logs
kubectl logs -f deployment/backend -n notes-app

# PostgreSQL logs
kubectl logs -f postgres-0 -n notes-app

# All logs
kubectl logs -f -l app=backend -n notes-app --all-containers=true
```

### Health Checks

```bash
# Check pod health
kubectl get pods -n notes-app

# Describe pod
kubectl describe pod <pod-name> -n notes-app

# Check events
kubectl get events -n notes-app --sort-by='.lastTimestamp'
```

### Database Operations

```bash
# Connect to PostgreSQL
kubectl exec -it postgres-0 -n notes-app -- psql -U postgres -d notes_app

# Backup database
kubectl exec postgres-0 -n notes-app -- pg_dump -U postgres notes_app > backup.sql

# Restore database
kubectl exec -i postgres-0 -n notes-app -- psql -U postgres notes_app < backup.sql
```

---

## 🐛 Troubleshooting

### Issue: ArgoCD Not Syncing

```
Application status: OutOfSync
```

**Check:**
```bash
# View application details
argocd app get notes-app

# View differences
argocd app diff notes-app

# Force sync
argocd app sync notes-app --force
```

### Issue: Pods in CrashLoopBackOff

```
NAME                  READY   STATUS             RESTARTS
backend-xxx-yyy       0/1     CrashLoopBackOff   5
```

**Debug:**
```bash
# Check logs
kubectl logs backend-xxx-yyy -n notes-app

# Check pod details
kubectl describe pod backend-xxx-yyy -n notes-app

# Common causes:
# - Database connection failure
# - Missing environment variables
# - Image pull errors
```

### Issue: LoadBalancer Stuck in Pending

```
EXTERNAL-IP: <pending>
```

**Solutions:**
```bash
# Check service
kubectl describe svc frontend-service -n notes-app

# Verify AWS LB controller
kubectl get pods -n kube-system | grep aws-load-balancer

# Check EKS tags on subnets
aws ec2 describe-subnets --filters "Name=tag:kubernetes.io/role/elb,Values=1"
```

### Issue: Database Connection Failed

```
Error: dial tcp: lookup postgres-service: no such host
```

**Fix:**
```bash
# Check PostgreSQL service
kubectl get svc -n notes-app postgres-service

# Check PostgreSQL pod
kubectl get pods -n notes-app | grep postgres

# Test connection from backend pod
kubectl exec -it backend-xxx-yyy -n notes-app -- nc -zv postgres-service 5432
```

### Issue: Out of Disk Space

```
Pod evicted due to disk pressure
```

**Solution:**
```bash
# Check PVC usage
kubectl get pvc -n notes-app

# Increase PVC size (edit manifest)
storage: 20Gi  # Increase from 10Gi

# Apply change
git commit -am "Increase PostgreSQL storage to 20Gi"
git push
```

### Debug Commands

```bash
# Get all resources
kubectl get all -n notes-app

# View ArgoCD application
kubectl get application -n argocd notes-app -o yaml

# Check sync status
argocd app get notes-app --refresh

# View pod events
kubectl get events -n notes-app --field-selector involvedObject.name=<pod-name>

# Execute command in pod
kubectl exec -it <pod-name> -n notes-app -- /bin/sh
```

---

## 📊 Monitoring

### Application Health

```bash
# ArgoCD health status
argocd app get notes-app | grep Health

# Pod readiness
kubectl get pods -n notes-app -o wide

# Service endpoints
kubectl get endpoints -n notes-app
```

### Resource Usage

```bash
# CPU and memory usage
kubectl top pods -n notes-app

# Node resources
kubectl top nodes
```

---

## 🔒 Security Best Practices

✅ **Secrets Management**: Never commit plain-text passwords  
✅ **RBAC**: Use least-privilege access  
✅ **Network Policies**: Restrict pod-to-pod communication  
✅ **Image Scanning**: Use signed/scanned images only  
✅ **Resource Limits**: Set memory/CPU limits on all pods  

---

## 🔗 Related Repositories

- **Source Code**: [MIND](https://github.com/who-sam/MIND)
- **Infrastructure**: [mind-infra-pipeline](https://github.com/who-sam/mind-infra-pipeline)
- **CI Pipeline**: [mind-ci-pipeline](https://github.com/who-sam/mind-ci-pipeline)
- 

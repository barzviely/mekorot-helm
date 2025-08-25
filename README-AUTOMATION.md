# 🤖 Complete "All From Code" GitOps Setup

This guide shows how to set up **complete GitOps automation** without any manual `kubectl apply` commands.

## 🎯 **Architecture: App of Apps Pattern**

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   Terraform     │    │   App of Apps    │    │  Applications   │
│   (Bootstrap)   │───▶│   (GitOps Root)  │───▶│   (Managed)     │
└─────────────────┘    └──────────────────┘    └─────────────────┘
│                      │                      │
├─ EKS Cluster        ├─ ArgoCD Self-Mgmt   ├─ sample-app
├─ ArgoCD (Basic)     ├─ SSH Secret (ESO)   ├─ sample-app-1
├─ External Secrets   ├─ Bootstrap Secret   ├─ [future apps]
└─ Initial IRSA       └─ All Applications   └─ Auto-discovery
```

## 🚀 **"All From Code" Setup Steps**

### **Step 1: Store SSH Key in AWS Secrets Manager**

```bash
# Store your SSH private key in AWS Secrets Manager
aws secretsmanager create-secret \
  --name "argocd/ssh-private-key" \
  --description "ArgoCD SSH private key for GitOps repository access" \
  --secret-string '{"private-key":"'$(cat ~/.ssh/id_ed25519 | tr '\n' '\\n')'"}'
```

### **Step 2: Deploy Infrastructure (Terraform)**

```bash
cd /path/to/mekorot-terraform/envs/central-workload
terraform apply -var-file=../common/common.auto.tfvars -var-file=central-workload.tfvars
```

This creates:
- ✅ EKS cluster with ArgoCD
- ✅ External Secrets Operator
- ✅ IRSA for External Secrets
- ✅ Karpenter and all infrastructure tools

### **Step 3: Bootstrap GitOps (One-Time)**

```bash
# Apply ONLY the App of Apps - this manages everything else
kubectl apply -f https://raw.githubusercontent.com/barzviely/mekorot-helm/main/environments/central-workload/app-of-apps.yaml
```

### **Step 4: Verify Complete Automation**

```bash
# Check that App of Apps is managing all applications
kubectl get applications -n argo

# Verify SSH secret is created by External Secrets
kubectl get secret repo-mekorot-helm -n argo

# Check that applications are syncing
kubectl get applications -n argo -o wide
```

## ✅ **What This Achieves**

### **Terraform Manages:**
- 🏗️ Infrastructure (EKS, VPC, IAM)
- 🔧 Platform tools (ArgoCD, Karpenter, External Secrets)
- 🔐 IRSA for External Secrets Operator

### **GitOps Repository Manages:**
- 📦 ArgoCD configuration (self-management)
- 🔑 SSH secrets (via External Secrets Operator)
- 🚀 All applications (sample apps, future apps)
- 🎯 Complete application lifecycle

### **No Manual kubectl Commands:**
- ✅ SSH secrets managed by External Secrets Operator
- ✅ ArgoCD manages its own configuration
- ✅ Applications auto-discover and deploy
- ✅ Complete Infrastructure as Code

## 🔄 **GitOps Workflow**

1. **Developer pushes code** to `argocd-k8s-environments`
2. **ArgoCD detects changes** automatically
3. **Applications sync** without manual intervention
4. **External Secrets** pulls SSH keys from AWS
5. **Complete automation** - no human intervention needed

## 📁 **Repository Structure**

```
argocd-k8s-environments/
├── environments/central-workload/
│   ├── app-of-apps.yaml                    # 🎯 Root application (apply once)
│   ├── argocd/
│   │   ├── argocd.yaml                     # ArgoCD self-management
│   │   ├── bootstrap-secret.yaml          # SSH secret via ESO
│   │   └── values/values.yaml              # ArgoCD configuration
│   ├── sample-app/
│   │   ├── sample-app.yaml                 # Auto-managed
│   │   └── values/values.yaml
│   └── sample-app-1/
│       ├── sample-app-1.yaml               # Auto-managed
│       └── values/values.yaml
└── charts/mekorot/                         # Universal Helm chart
```

## 🎯 **Adding New Applications**

Simply add a new directory and ArgoCD will auto-discover it:

```bash
mkdir -p environments/central-workload/my-new-app/values
# Create application YAML and values
git commit && git push
# ArgoCD automatically deploys it!
```

## 🚨 **Security Benefits**

- 🔐 **SSH keys in AWS Secrets Manager** (encrypted, rotatable)
- 🔒 **IRSA authentication** (no static credentials)
- 📝 **Audit trail** via Git and AWS CloudTrail
- 🎯 **Least privilege** via precise IAM policies
- 🔄 **Immutable infrastructure** - everything in code

This approach gives you **complete GitOps automation** with **zero manual kubectl commands** while maintaining **enterprise security standards**.

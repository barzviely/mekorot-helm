# ArgoCD Applications with Mekorot Universal Helm Chart

This repository demonstrates a **scalable approach** for deploying multiple applications using ArgoCD with the Mekorot production-ready universal Helm chart.

## 🏗️ Architecture

Instead of creating separate Helm charts for each application, we use:
- **One Mekorot chart** (`charts/mekorot/`) - universal, production-ready chart for any application
- **Environment-specific values files** - customize behavior per app/environment  
- **ArgoCD Applications** - point to the Mekorot chart with specific values

## 📁 Directory Structure

```
argocd-k8s-environments/
├── charts/
│   └── mekorot/                       # ✅ Universal production-ready chart
│       ├── Chart.yaml
│       ├── values.yaml               # Default values with feature flags
│       └── templates/
│           ├── deployment.yaml       # Full-featured deployment
│           ├── svc.yaml             # Service with annotations
│           ├── ingress.yaml         # ALB/NLB ingress support
│           ├── configmap.yaml       # Configuration management
│           ├── externalSecret.yaml  # AWS Secrets Manager integration
│           ├── persistentvolume.yaml # EFS/EBS volumes
│           ├── persistentvolumeclaim.yaml
│           ├── storageclass.yaml    # Dynamic provisioning
│           ├── serviceaccount.yaml  # RBAC & IRSA
│           └── _helpers.tpl         # Template helpers
└── environments/
    └── central-workload/
        ├── sample-app/
        │   ├── sample-app.yaml       # ArgoCD Application
        │   └── values/
        │       └── values.yaml       # App-specific values
        └── sample-app-1/
            ├── sample-app-1.yaml     # ArgoCD Application  
            └── values/
                └── values.yaml       # App-specific values
```

## 🚀 Benefits of This Approach

### ✅ Scalable
- Add new apps by creating only a values file
- No code duplication
- Consistent deployment patterns

### ✅ Maintainable  
- Update chart once, affects all apps
- Centralized security and best practices
- Easy to add new features (monitoring, security contexts, etc.)

### ✅ Flexible
- Override any value per application
- Support different types of apps (web, API, worker, etc.)
- Environment-specific configurations

## 📝 Adding a New Application

To add a new application (e.g., `my-new-app`):

1. **Create values file:**
   ```bash
   mkdir -p environments/central-workload/my-new-app/values
   ```

2. **Create `environments/central-workload/my-new-app/values/values.yaml`:**
   ```yaml
   nameOverride: "my-new-app"
   
   app:
     deployment:
       enabled: true
       name: "my-new-app"
       
     image:
       repository: myregistry/my-new-app
       tag: "v1.0.0"
     
     resources:
       requests:
         cpu: 100m
         memory: 256Mi
       limits:
         cpu: 500m
         memory: 512Mi
     
     nodeSelector:
       type: default
       
     service:
       enabled: true
       name: "my-new-app-service"
       port: 80
       
     # Enable features as needed
     ingress:
       enabled: true
       hosts:
         - host: my-new-app.domain.com
           paths:
             - path: /
               backend:
                 service:
                   name: my-new-app-service
                   
     externalSecret:
       enabled: true
       # AWS Secrets Manager integration
       
     config:
       enabled: true
       data:
         app.properties: |
           database.url=jdbc:postgresql://...
   ```
   ```

3. **Create ArgoCD Application:**
   ```yaml
   apiVersion: argoproj.io/v1alpha1
   kind: Application
   metadata:
     name: my-new-app
     namespace: argo
   spec:
     sources:
       - repoURL: https://github.com/your-repo/argocd-k8s-environments.git
         path: charts/mekorot
         helm:
           valueFiles:
             - $values/environments/central-workload/my-new-app/values/values.yaml
       - repoURL: https://github.com/your-repo/argocd-k8s-environments.git
         ref: values
     destination:
       namespace: my-new-app
   ```

4. **Deploy:** `kubectl apply -f environments/central-workload/my-new-app/`

## 🔧 Mekorot Chart Features ⭐

The Mekorot chart is production-ready and supports:

### 🚀 **Core Resources**
- **Deployments** with advanced configuration
- **Services** with cloud provider annotations  
- **Ingress** with ALB/NLB support (internal/external)
- **ConfigMaps** for application configuration
- **External Secrets** (AWS Secrets Manager integration)

### 💾 **Storage & Persistence**  
- **PersistentVolumes** (EFS, EBS)
- **PersistentVolumeClaims** with dynamic provisioning
- **StorageClasses** for different storage types
- **EFS volume mounting** for shared storage

### 🔐 **Security & RBAC**
- **ServiceAccounts** with IRSA support
- **Pod security contexts** and constraints
- **RBAC** integration
- **Secret management** via External Secrets Operator

### 📈 **Scaling & Performance**
- **Horizontal Pod Autoscaler** support
- **Resource limits & requests**
- **Node selectors, affinity, and tolerations**
- **Health checks** (readiness, liveness, startup)

### 🎛️ **Feature Flags**
All features can be enabled/disabled via `app.***.enabled` flags:
```yaml
app:
  deployment:
    enabled: true
  service:  
    enabled: true
  ingress:
    enabled: false
  externalSecret:
    enabled: true
  persistentVolume:
    enabled: true
```

## 🎯 Real Example: 15 Applications

With this approach, deploying 15 applications means:
- ✅ **1 Mekorot chart** (production-ready, maintained once)
- ✅ **15 values files** (app-specific configs)
- ✅ **15 ArgoCD apps** (pointing to same chart)

Total maintenance: **1 universal chart + 15 small values files** instead of **15 complete charts**!

### 🎯 **Why Mekorot Chart?**
- **Battle-tested** in production environments
- **Feature-complete** with everything you need
- **Cloud-native** with AWS integrations (ALB, EFS, Secrets Manager)
- **Flexible** feature flags for any use case
- **Secure** with built-in best practices
- **Terraform-integrated** - works with your existing TF setup
# mekorot-helm

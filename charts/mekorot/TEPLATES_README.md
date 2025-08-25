## 🗂️ Templates

The `templates/` directory holds the Kubernetes manifest templates that Helm will render and apply. Each file corresponds to a core Kubernetes resource in your chart:

- **`deployment.yaml`**  
  Defines the Deployment resource:
  - Specifies the container image, tag, and pull policy  
  - Sets replica count, resources (CPU/memory requests & limits)  
  - Configures environment variables, volume mounts, and probes

- **`service.yaml`**  
  Creates a Service to expose your pods:
  - Controls how pods are reachable (ClusterIP, NodePort, LoadBalancer)  
  - Maps Service ports to container ports  
  - Supports annotations for cloud-provider integrations (e.g., ALB or NLB)

- **`ingress.yaml`**  
  Defines an Ingress resource for HTTP/S routing:
  - Hosts, paths, and backend Service mappings  
  - TLS configuration (certificates, secrets)  
  - Ingress-controller annotations (e.g., path rewriting, rate limits)

- **`configmap.yaml`**  
  Generates a ConfigMap for non-sensitive configuration data:
  - Application settings (e.g., flags, feature toggles)  
  - Can be consumed as environment variables or mounted volumes

- **`persistentvolume.yaml`**  
  Provisions a cluster-wide PersistentVolume:
  - References a StorageClass (e.g., EFS CSI driver)  
  - Defines capacity, access modes, and reclaim policy

- **`persistentvolumeclaim.yaml`**  
  Requests storage via a PVC:
  - Binds to a matching PersistentVolume or dynamically provisions one  
  - Uses size and storageClass parameters from `values.yaml`

- **`storageclass.yaml`**  
  Declares a StorageClass for dynamic provisioning:
  - Driver (e.g., `efs.csi.aws.com`), reclaim policy, and volume binding mode  
  - Allows fine-tuning parameters like throughput or performance tiers

- **`serviceaccount.yaml`**  
  Creates a ServiceAccount for your pods:
  - Namespaced identity for RBAC  
  - Can be linked to IAM roles via IRSA (on EKS)

- **`externalSecret.yaml`**  
  (When using External Secrets) Defines an ExternalSecret to fetch secrets from AWS Secrets Manager:
  - Maps external secret keys to Kubernetes Secret data  
  - Automates secret synchronization

- **`_helpers.tpl`**  
  Contains reusable template helpers and named snippets:
  - Functions for labels, annotations, fullname logic, etc.  
  - Keeps templates DRY by centralizing common patterns

- **`NOTES.txt`**  
  Post-install/uninstall reminders and usage hints:
  - Printed after a successful release to guide next steps  
  - Can include sample `kubectl` or `helm` commands

Each template references values from your `values.yaml`, allowing you to customize every aspect of the deployed resources without editing the manifests directly.

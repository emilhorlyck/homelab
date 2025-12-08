# Homelab

## Minikube

### Install Minikube

```bash
brew install minikube
```

### Setup Minikube

```bash
minikube start
```

## ArgoCD

### Install ArgoCD

Install ArgoCD in the argocd namespace.

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### Verify ArgoCD

```bash
kubectl get all -n argocd
```

### Port Forwarding

This will forward the ArgoCD server to your local machine on port 8080.

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Get the password with this command

```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```

visit the webinterface on:´localhost:8080´

## Accessing Services via Traefik

### Minikube Tunnel

Minikube doesn't have a built-in LoadBalancer controller, so LoadBalancer services will remain in `<pending>` state. To expose services properly, use `minikube tunnel`:

```bash
minikube tunnel
```

**Note:**
- This command requires sudo/admin privileges
- Keep this terminal running in the background
- It will assign an EXTERNAL-IP to the Traefik LoadBalancer service

### Configure Local DNS

Once the tunnel is running, get the Traefik LoadBalancer IP:

```bash
kubectl get svc -n traefik traefik
```

The EXTERNAL-IP will typically be `127.0.0.1` when using minikube tunnel.

Add the following entries to your `/etc/hosts` file:

```bash
sudo nano /etc/hosts
```

Add these lines (replace `<EXTERNAL-IP>` with the actual IP from above):

```
<EXTERNAL-IP>  grafana.local argocd.local
```

### Access Services

After configuring DNS, you can access services using their configured hostnames:

- **Grafana**: `https://grafana.local`
- **ArgoCD**: `https://argocd.local`

**Note:** You may see SSL certificate warnings since we're using self-signed certificates. This is normal for local development.

## App of Apps Pattern

This repository uses ArgoCD's [App of Apps](https://argo-cd.readthedocs.io/en/stable/operator-manual/cluster-bootstrapping/#app-of-apps-pattern) pattern to manage multiple applications declaratively.

### Structure

The `apps/` folder contains ArgoCD `Application` YAML definitions. Application definitions can be organized directly in the `apps/` folder or nested in subdirectories (e.g., `apps/infrastructure/`, `apps/observability/`). Each YAML file containing an `Application` resource represents a separate application that ArgoCD will deploy and manage.

The root `app-of-apps.yaml` file is the parent Application that points to the `apps/` directory. When ArgoCD syncs this parent application, it automatically discovers and manages all Application manifests found recursively within the `apps/` folder and its subdirectories.

### Apps Folder Requirements

Each YAML file in the `apps/` folder (or its subdirectories) must be a valid ArgoCD `Application` resource with:

- `apiVersion: argoproj.io/v1alpha1`
- `kind: Application`
- `metadata.name`: Unique name for the application
- `metadata.namespace`: Should be `argocd` (where Applications are managed)
- `spec.source`: Defines where the application manifests/charts are located (Helm chart repository, Git repository, etc.)
- `spec.destination`: Defines where the application should be deployed (cluster and namespace)

### Example Application Definitions

Applications can reference:

- **Helm Charts** from chart repositories (e.g., `cert-manager.yaml`, `lgtm-distributed.yaml`)
- **Git repositories** with Kubernetes manifests (e.g., `guestbook.yaml`)
- **Local paths** within this repository

### External Resources

- [ArgoCD App of Apps Pattern Documentation](https://argo-cd.readthedocs.io/en/stable/operator-manual/cluster-bootstrapping/#app-of-apps-pattern)
- [ArgoCD Application Specification](https://argo-cd.readthedocs.io/en/stable/operator-manual/application.yaml/)
- [ArgoCD Application Concepts](https://argo-cd.readthedocs.io/en/stable/core_concepts/)

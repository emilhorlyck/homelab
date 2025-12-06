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

## App of Apps Pattern

This repository uses ArgoCD's [App of Apps](https://argo-cd.readthedocs.io/en/stable/operator-manual/cluster-bootstrapping/#app-of-apps-pattern) pattern to manage multiple applications declaratively.

### Structure

The `apps/` folder contains ArgoCD `Application` YAML definitions. Each file in this directory represents a separate application that ArgoCD will deploy and manage.

The root `app-of-apps.yaml` file is the parent Application that points to the `apps/` directory. When ArgoCD syncs this parent application, it automatically discovers and manages all Application manifests found in the `apps/` folder.

### Apps Folder Requirements

Each file in the `apps/` folder must be a valid ArgoCD `Application` resource with:

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

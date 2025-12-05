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

# ArgoCD Installation

This section describes how **ArgoCD v3.3.0** was installed on the Kubernetes Home Lab
using the official vanilla manifest.

ArgoCD is a **declarative GitOps continuous delivery tool** for Kubernetes.
It monitors a Git repository and automatically syncs the desired state to the cluster.

--- 

## Prerequisites

- A running Kubernetes cluster
- `kubectl` configured with access to the cluster
- Sufficient permissions to create namespaces and cluster-wide resources

---

## Installation

### Create the namespace

ArgoCD runs in its own dedicated namespace:

```bash
kubectl create namespace argocd
```

---

### Apply the official manifest

The installation uses the official upstream manifest for **ArgoCD v3.3.0**:

```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v3.3.0/manifests/install.yaml
```

!!! note
This installs ArgoCD with all default components.
No Helm chart or Kustomize overlay is used.

---

### Verify the installation

Wait for all pods to be in `Running` state:

```bash
kubectl get pods -n argocd
```

Expected output:

```
NAME                                                  READY   STATUS    RESTARTS   AGE
argocd-application-controller-0                       1/1     Running   0          ...
argocd-dex-server-xxxx                                1/1     Running   0          ...
argocd-notifications-controller-xxxx                  1/1     Running   0          ...
argocd-redis-xxxx                                     1/1     Running   0          ...
argocd-repo-server-xxxx                               1/1     Running   0          ...
argocd-server-xxxx                                    1/1     Running   0          ...
```

---

## Components Installed

| Component                        | Role                                              |
|----------------------------------|---------------------------------------------------|
| `argocd-server`                  | Main API server and Web UI                        |
| `argocd-application-controller`  | Reconciles cluster state with Git                 |
| `argocd-repo-server`             | Clones and renders Git repositories               |
| `argocd-dex-server`              | Handles SSO and authentication                    |
| `argocd-redis`                   | Caching layer                                     |
| `argocd-notifications-controller`| Sends notifications on sync events                |

!!! info
All services are of type **ClusterIP**.
External access is handled separately via Gateway API and Cloudflare Tunnel.

---

## Design Decisions

- **Vanilla install** — no Helm, no Kustomize, keeping the setup simple and auditable
- **ClusterIP only** — no LoadBalancer or NodePort, external exposure is managed at the Gateway layer
- **v3.3.0** — pinned version for reproducibility

---
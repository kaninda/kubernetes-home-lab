# GitOps Workflow

This section describes the **GitOps workflow** used in this lab to deploy
and manage Kubernetes workloads via ArgoCD.

The core principle is simple: **Git is the single source of truth**.
Any change to the cluster must go through a Git commit.

---

## Core Concept

![overview](diagrams/gitops-workflow.svg){ width="40%" }


- **No manual `kubectl apply`** for application deployments
- ArgoCD continuously reconciles the **desired state** (Git) with the **actual state** (cluster)
- Any drift is detected and can be corrected automatically

---

## Workflow

### 1. Define the desired state in Git

All Kubernetes manifests (Deployments, Services, ConfigMaps, HTTPRoutes, etc.)
are stored in a Git repository.

```
kubernetes-home-lab/
├── apps/
│   ├── my-app/
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   └── httproute.yaml
```

---

### 2. Create an ArgoCD Application

An **Application** resource tells ArgoCD where to find the manifests
and where to deploy them:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/kaninda/kubernetes-home-lab
    targetRevision: HEAD
    path: apps/my-app
  destination:
    server: https://kubernetes.default.svc
    namespace: my-app
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

Apply it with:

```bash
kubectl apply -f apps/my-app/application.yaml
```

---

### 3. ArgoCD syncs the cluster

ArgoCD detects the new Application and reconciles the cluster state:

- Pulls the manifests from the Git repository
- Applies them to the target namespace
- Reports the sync status in the UI

!!! tip
With `automated` sync policy enabled, ArgoCD will automatically apply
any new commit to the target branch — no manual intervention needed.

---

## Sync Policies

| Policy        | Behavior                                              |
|---------------|-------------------------------------------------------|
| `automated`   | Automatically syncs on every Git change               |
| `prune: true` | Deletes resources removed from Git                    |
| `selfHeal: true` | Reverts manual changes made directly to the cluster|

!!! warning
With `selfHeal: true`, any manual `kubectl apply` or `kubectl delete`
will be reverted by ArgoCD on the next reconciliation loop.

---

## Verify Sync Status

Check the application status via CLI:

```bash
kubectl get application -n argocd
```

Or open the ArgoCD UI at `https://argocd.kanismile.com` to visualize
the sync status, resource tree, and live diff.

---

## Design Decisions

- **Automated sync** — reduces manual operations and enforces GitOps discipline
- **Self-healing** — prevents configuration drift
- **Prune enabled** — keeps the cluster clean when resources are removed from Git
- **Single repo** — all manifests live in the same repository as the documentation

---

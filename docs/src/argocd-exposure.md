# ArgoCD Exposure via Gateway

This section describes how **ArgoCD** is exposed externally using **Gateway API**
and **Cloudflare Tunnel**.

The ArgoCD UI is accessible at `argocd.kanismile.com` without any LoadBalancer
or open inbound port on the cluster.

---

## Architecture

Traffic follows the same pattern as all other services in this lab:

```
User → Cloudflare DNS → Cloudflare Tunnel → Gateway API → HTTPRoute → argocd-server
```

- The `argocd-server` service remains **ClusterIP**
- The **NGINX Gateway Fabric** handles routing via the `edge-gateway`
- **Cloudflare Tunnel** provides secure external access

---

## Flow Diagram

![overview](diagrams/argocd-exposure-noir.svg){ width="100%" }

---

## HTTPRoute

The HTTPRoute is deployed in the `argocd` namespace and references the shared
`edge-gateway` in the `nginx-gateway` namespace.

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: argocd
  namespace: argocd
spec:
  parentRefs:
    - name: edge-gateway
      namespace: nginx-gateway
  hostnames:
    - argocd.kanismile.com
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /
      backendRefs:
        - name: argocd-server
          port: 80
```

Apply it with:

```bash
kubectl apply -f httproute-argocd.yaml
```

---

## Verify the Route

Check the HTTPRoute status:

```bash
kubectl get httproute argocd -n argocd
```

Expected output:

```
NAME     HOSTNAMES                    AGE
argocd   ["argocd.kanismile.com"]     ...
```

Verify it is accepted by the gateway controller:

```bash
kubectl describe httproute argocd -n argocd
```

Look for:

```
Conditions:
  Type: Accepted    Status: True
  Type: ResolvedRefs  Status: True
```

---

## Design Decisions

- **No Ingress** — Gateway API is used exclusively across the lab
- **ClusterIP preserved** — the `argocd-server` service is not modified
- **Shared gateway** — the `edge-gateway` is reused across all services, avoiding resource duplication
- **No TLS at the cluster level** — TLS is terminated at the Cloudflare edge

!!! info
The `edge-gateway` is defined in the `nginx-gateway` namespace and is shared
across all HTTPRoutes in the lab. See the [NGINX Gateway Fabric](nginx-gateway.md) section for details.

---

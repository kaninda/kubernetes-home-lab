# NGINX Gateway Fabric

This section describes how **NGINX Gateway Fabric v2.4.1** was installed
and configured as the Gateway API implementation in this lab.

NGINX Gateway Fabric acts as the **data plane** for the Gateway API.
It translates `Gateway` and `HTTPRoute` resources into NGINX routing configuration,
and handles all inbound HTTP traffic coming from the Cloudflare Tunnel.

---

## Architecture

![NGINX Gateway Fabric Architecture](diagrams/nginx-gateway-1.svg){ width="100%" }

---

Two deployments run in the `nginx-gateway` namespace:

| Component             | Image                                           | Role                              |
|-----------------------|-------------------------------------------------|-----------------------------------|
| `nginx-gateway`       | `ghcr.io/nginx/nginx-gateway-fabric:2.4.1`      | Controller — watches Gateway API resources |
| `edge-gateway-nginx`  | `ghcr.io/nginx/nginx-gateway-fabric/nginx:2.4.1`| Data plane — handles HTTP traffic |

The controller continuously reconciles `GatewayClass`, `Gateway`, and `HTTPRoute`
resources and configures the NGINX data plane accordingly.

---

## Installation

NGINX Gateway Fabric is installed using the **official upstream manifests** (no Helm).


### 1. Install the Gateway API CRDs

```bash
kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.2.0/standard-install.yaml
```

### 2. Install NGINX Gateway Fabric

```bash
kubectl apply -f https://raw.githubusercontent.com/nginx/nginx-gateway-fabric/v2.4.1/deploy/crds.yaml
kubectl apply -f https://raw.githubusercontent.com/nginx/nginx-gateway-fabric/v2.4.1/deploy/default/deploy.yaml
```

!!! note
This installs NGINX Gateway Fabric in the `nginx-gateway` namespace
with all default settings.

### 3. Verify the installation

```bash
kubectl get pods -n nginx-gateway
```

Expected output:

```
NAME                                  READY   STATUS    RESTARTS   AGE
nginx-gateway-xxxx                    1/1     Running   0          ...
edge-gateway-nginx-xxxx               1/1     Running   0          ...
```

---

## GatewayClass

The `GatewayClass` resource registers NGINX Gateway Fabric as a Gateway API controller:

```bash
kubectl get gatewayclass
```

Expected output:

```
NAME    CONTROLLER                                   ACCEPTED   AGE
nginx   gateway.nginx.org/nginx-gateway-controller   True       ...
```

The GatewayClass is automatically created by the install manifest and links
to the `nginx-gateway-proxy-config` NginxProxy configuration.

---

## Gateway

A single shared **Gateway** named `edge-gateway` is deployed in the `nginx-gateway` namespace.
It serves as the unique entry point for all HTTP traffic in the lab.

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: edge-gateway
  namespace: nginx-gateway
spec:
  gatewayClassName: nginx
  listeners:
    - name: http
      protocol: HTTP
      port: 80
      allowedRoutes:
        namespaces:
          from: All
```

Apply it with:

```bash
kubectl apply -f nginx_gateway_fabric/gateway.yaml
```

Key design choices:

- **Single listener** on port `80` (HTTP)
- **`from: All`** — accepts HTTPRoutes from any namespace in the cluster
- **Shared gateway** — all services reuse this single entry point

---

## Service Exposure

The `edge-gateway-nginx` service is exposed as a **NodePort**:

```
edge-gateway-nginx   NodePort   10.103.92.37   80:31224/TCP
```

The Cloudflare Tunnel (`cloudflared`) targets this NodePort to forward
external traffic into the cluster.

!!! info
No LoadBalancer or public IP is required.
The NodePort is only reachable internally — Cloudflare Tunnel handles
the secure external connection.

---

## Verify the Gateway

Check the Gateway status:

```bash
kubectl get gateway edge-gateway -n nginx-gateway
```

Check attached routes:

```bash
kubectl get httproute -A
```

Expected — 4 routes currently attached to the `edge-gateway`:

```
NAMESPACE    NAME           HOSTNAMES                        AGE
argocd       argocd         ["argocd.kanismile.com"]         ...
monitoring   grafana        ["grafana.kanismile.com"]        ...
monitoring   prometheus     ["prometheus.kanismile.com"]     ...
monitoring   alertmanager   ["alertmanager.kanismile.com"]   ...
```

---

## Design Decisions

- **Manifest install over Helm** — keeps the setup explicit and auditable
- **Single shared Gateway** — avoids duplicating listener config per service
- **NodePort over LoadBalancer** — no cloud provider or MetalLB required
- **`from: All`** — allows HTTPRoutes to be defined close to each application's namespace

---


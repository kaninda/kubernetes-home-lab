# Logical Architecture

This section describes the **logical architecture** of the Kubernetes Home Lab,
focusing on traffic flow, component responsibilities, and design decisions.

Unlike the physical topology, this view focuses on **how components interact**
to expose services from inside the cluster to external users.

---

## Architecture Overview

The platform is designed to expose applications securely **without requiring
public inbound connectivity**.

Instead of exposing services via public IPs or LoadBalancers, this architecture
relies on **Cloudflare Tunnel** and **Gateway API**.

![overview](diagrams/logical-architecture.svg){ width="40%" }

---

## High-Level Flow

External traffic follows this path:

**User → Cloudflare DNS → Cloudflare Tunnel → Gateway API → HTTPRoute → Service → Pod**

---

## Components

### Cloudflare DNS

- Public entry point
- Resolves application domain (e.g. argocd.example.com)
- Managed externally via Cloudflare

---

### Cloudflare Tunnel (cloudflared)

- Runs inside the cluster as a Deployment
- Establishes **outbound-only connections** to Cloudflare
- Eliminates the need for:
    - public IP addresses
    - port forwarding
    - firewall exposure

➡️ This significantly improves security by avoiding inbound traffic.

---

### Gateway API

Gateway API provides **Kubernetes-native traffic routing**.

It replaces traditional Ingress with a more flexible model.

Main resources:

- **GatewayClass** → defines the controller
- **Gateway** → entry point (listeners, ports)
- **HTTPRoute** → routing rules

---

### NGINX Gateway Fabric

- Implements Gateway API
- Acts as the **data plane**
- Routes HTTP traffic to services

Responsibilities:

- TLS termination (optional)
- Host-based routing
- Path-based routing

---

### HTTPRoute

Defines how traffic is routed:

Example:

```yaml
hostnames:
  - argocd.example.com
```

Routes requests to:
- Service: argocd-server
- Port: 80

---

### Services

- Expose Kubernetes workloads internally
- Usually of type ClusterIP
- Gateway handles external access

---

### Pods

- Actual application workloads
- Example: ArgoCD server

## Traffic Flow (Detailed)

**User → Cloudflare DNS → Cloudflare Tunnel → Gateway API → HTTPRoute → Service → Pod**

1. The user sends a request to: https://argocd.example.com
2. The domain is resolved by **Cloudflare DNS**

3. The request is routed through the **Cloudflare edge network**

4. Cloudflare forwards the request to the **Cloudflare Tunnel**

5. The **cloudflared** agent (running in the cluster) receives the request

6. The request is forwarded to the **Gateway API listener**

7. The **HTTPRoute** matches the hostname

8. The request is forwarded to the corresponding **Service**

9. The Service forwards traffic to the **Pod**

---

---

## Design Decisions

### No LoadBalancer

This lab does not rely on cloud LoadBalancers or MetalLB.

External exposure is handled via **Cloudflare Tunnel**.

---

### Gateway API instead of Ingress

Gateway API is used instead of traditional Ingress because:

- More expressive routing model
- Better separation of concerns
- Future Kubernetes standard

---

### Outbound-only connectivity

No inbound ports are opened on the network.

All connections are initiated from inside the cluster via cloudflared.

This improves security by reducing the attack surface.

---

### Separation of concerns

| Layer              | Responsibility              |
|-------------------|---------------------------|
| Cloudflare        | DNS + Edge routing        |
| cloudflared       | Secure tunnel             |
| Gateway API       | Routing model             |
| NGINX Gateway     | Traffic processing        |
| Kubernetes        | Workload execution        |

---

## Security Model

- No public IP exposure on cluster nodes
- No open ports on the router
- All traffic is initiated from inside the cluster
- TLS is terminated at Cloudflare edge (optional)

This model significantly reduces the attack surface compared to traditional setups.

---

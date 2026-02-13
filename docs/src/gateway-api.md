# Gateway API

This section describes the installation and configuration of **Gateway API**
used to expose applications inside the Kubernetes cluster.

Gateway API provides a **modern, flexible alternative to Ingress**, offering
a more expressive routing model and better separation of concerns.

---

## Overview

Gateway API introduces a set of Kubernetes resources to manage traffic:

- **GatewayClass** → defines the controller implementation
- **Gateway** → defines entry points (listeners, ports)
- **HTTPRoute** → defines routing rules

This model separates:

- infrastructure (Gateway)
- application routing (HTTPRoute)

---

## Why Gateway API

Gateway API was chosen over traditional Ingress because:

- More flexible routing model
- Better separation of responsibilities
- Improved support for multi-team environments
- Considered the future of Kubernetes traffic management

---

## Architecture

Gateway API operates with:

- a **controller** (NGINX Gateway Fabric)
- a **GatewayClass**
- one or more **Gateways**
- multiple **HTTPRoutes**

**GatewayClass → Gateway → HTTPRoute → Service → Pod**

---



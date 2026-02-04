# Overview

This documentation describes the design, setup, and operation of a **bare-metal Kubernetes home lab**.

The cluster is built using **kubeadm** on **Ubuntu Server**, with a strong focus on:
- clarity
- reproducibility
- production-like behavior

While this is a home lab, all design choices aim to stay as close as possible to
real-world Kubernetes environments.

---

## Cluster goals

The main goals of this Kubernetes Home Lab are:

- Learn Kubernetes fundamentals on bare metal
- Understand cluster internals and networking
- Practice installation, upgrades and troubleshooting
- Experiment with CNI, storage and GitOps tools
- Maintain a simple but realistic setup

---

## High-level architecture

The cluster consists of:
- one dedicated control plane node
- multiple worker nodes
- a flat Layer 2 home LAN
- static IP addressing or DHCP reservations
- a dedicated Pod network

All nodes run:
- Ubuntu Server
- containerd as container runtime
- kubeadm-managed Kubernetes components

---

## Design principles

This cluster follows a few key design principles:

- **Simplicity over complexity**
- **Predictability over automation**
- **Reproducibility over convenience**
- **Documentation-first approach**

Each component is installed and configured explicitly to ensure full understanding
of what happens under the hood.

---

## Scope and limitations

This home lab intentionally has the following limitations:

- Single control plane (no HA)
- No external load balancer
- No cloud provider integration
- No managed services

These limitations keep the cluster easy to reason about while still allowing
meaningful Kubernetes experimentation.

---

## Documentation structure

The documentation is organized as follows:

1. Hardware description reminds the physical foundation
2. Network topology explains physical and logical connectivity
3. OS preparation ensures all nodes are Kubernetes-ready
4. Kubernetes installation describes kubeadm setup
5. CNI configuration focuses on pod networking
6. Validation ensures cluster health and correctness

Each section builds on the previous one and should be followed in order.

# Network Topology

This section describes the **physical and logical network topology**
of the Kubernetes Home Lab.

The goal is to clearly understand:
- how nodes are connected at the network level
- how Kubernetes components communicate
- how Pod networking is handled

---
## High-level overview

The cluster is deployed on a **home LAN** with:
- a router providing DHCP reservations
- a dedicated 2.5 GbE switch
- all Kubernetes nodes connected via Ethernet

All nodes use **static IPs or DHCP reservations** to ensure
predictable addressing.

---
## Physical network topology

![Physical network topology (Calico)](diagrams/physical-network-topology.svg){ width="80%" }
---

## Kubernetes control plane communication

- The kube-apiserver runs on the control plane node
- Worker nodes communicate exclusively with the API server
- All control traffic uses the LAN network

---

## Pod network (Calico)

The cluster uses Calico as the CNI.
- Each node is assigned a Pod CIDR
- Pods communicate directly across nodes
- Pod traffic is encapsulated and routed by Calico
- The Pod CIDR does not overlap with the LAN network

![Physical-topology (Calico)](diagrams/physical-topology.svg){ width="30%" }



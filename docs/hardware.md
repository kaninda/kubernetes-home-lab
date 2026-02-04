# Hardware

This section describes the physical hardware used to build the Kubernetes Home Lab.

The goal is to use **compact, power-efficient, and affordable machines** while keeping
a setup close to a real-world bare-metal Kubernetes cluster.

## Hardware overview

| Node      | Role           | CPU                | RAM  | Storage | Network |
|-----------|----------------|--------------------|------|---------|---------|
| cp-1      | Control Plane  | Ryzen 5 3500U      | 16GB | 512GB SSD | 2.5 GbE |
| worker-1  | Worker Node    | Ryzen 5 5500U      | 32GB | 500GB SSD | 2.5 GbE |
| worker-2  | Worker Node    | Ryzen 5 5500U      | 32GB | 500GB SSD | 2.5 GbE |

## Control Plane node

The control plane is hosted on a dedicated mini-PC.

Characteristics:
- Sufficient CPU power for API server, scheduler and controller manager
- Moderate memory footprint (16 GB is sufficient for a single control plane)
- SSD storage for etcd and system components
- Dedicated Ethernet interface

This node is intentionally not used for application workloads.

## Worker nodes

Worker nodes are more powerful and designed to run application workloads.

Characteristics:
- Higher CPU core count for parallel workloads
- More memory for pods and caches
- Fast local SSD storage
- Wired Ethernet connectivity

This allows realistic scheduling, resource limits and scaling experiments.

## Networking hardware

All nodes are connected using a dedicated **2.5 GbE Ethernet switch**.

Design choices:
- Wired network only (no Wi-Fi)
- Predictable latency and throughput
- No network bottlenecks between nodes

## Key hardware design decisions

- Single control plane (home lab scope)
- Dedicated nodes (no virtualization)
- Wired Ethernet only
- Over-provisioned worker nodes
- Energy-efficient hardware






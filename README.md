# Kubernetes Home Lab

This repository documents the design, installation and operation of a
bare-metal Kubernetes cluster built at home using kubeadm.

Documentation, which is also a work in progress and a bit lower priority for now,
is available here: https://kaninda.github.io/kubernetes-home-lab/.

## Goals
- Build a production-like Kubernetes cluster
- Practice CKA / CKAD concepts
- Document each step clearly and reproducibly

## Cluster Overview
- Kubernetes version: 1.34.x
- OS: Ubuntu Server
- Runtime: containerd
- CNI: Calico
- Installation method: kubeadm

```mermaid
flowchart TB
    subgraph LAN["Home LAN (192.168.1.0/24)"]
        R[Router / DHCP<br/>IP reservations]
        S[2.5GbE Switch]
    end

    subgraph K8S["Kubernetes Cluster"]
        CP["cp-1<br/>Control Plane<br/>192.168.1.53"]
        W1["worker-1<br/>Worker Node<br/>192.168.1.55"]
        W2["worker-2<br/>Worker Node<br/>192.168.1.56"]
    end

    R --> S
    S --> CP
    S --> W1
    S --> W2

    CP ---|kube-apiserver| W1
    CP ---|kube-apiserver| W2

    subgraph PODS["Pod Network (Calico)"]
        P1["Pod"]
        P2["Pod"]
        P3["Pod"]
    end

    W1 --> P1
    W2 --> P2
    W2 --> P3
```

## Hardware
| Node | Role | CPU | RAM | Storage |
|-----|-----|-----|-----|---------|
| cp-1 | Control Plane | Ryzen 5 3500U | 16 GB | 512 GB SSD |
| worker-1 | Worker | Ryzen 5 5500U | 32 GB | 500 GB SSD |
| worker-2 | Worker | Ryzen 5 5500U | 32 GB | 500 GB SSD |

## Network
- LAN: 192.168.1.0/24
- Pod CIDR: 10.244.0.0/16
- All nodes connected via 2.5GbE switch

## Documentation
Step-by-step documentation is available in the `docs/` directory.

## Status
🚧 Work in progress

## Run mkdocs in local 

```bash
python -m mkdocs serve
```
## Deploy mkdocs to github page
```bash
python -m mkdocs gh-deploy
```

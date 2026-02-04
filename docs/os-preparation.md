# OS Preparation (Ubuntu Server)

This section describes all **system-level prerequisites** required before
installing Kubernetes using **kubeadm**.

The goal is to ensure:

- system stability
- Kubernetes compatibility
- predictable and reproducible behavior across all nodes

All steps below **must be applied on every node**:

- control plane
- worker nodes

---

## Base assumptions

- OS: **Ubuntu Server 22.04 LTS** (or newer)
- Architecture: **amd64**
- Network: static IP or DHCP reservation
- User has sudo privileges

---

## Disable swap

Kubernetes requires swap to be disabled.  
The **kubelet will refuse to start** if swap is enabled.

### Disable swap immediately

```bash
sudo swapoff -a
```
### Disable swap permanently

Edit /etc/fstab and comment out any swap entry:
```bash
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```

### Verify swap status

```bash
free -h
```
Expected output: Swap: 0B

---

## Load required kernel modules
Kubernetes networking requires specific kernel modules.

### Configure modules to load at boot
```bash
sudo tee /etc/modules-load.d/k8s.conf <<EOF
overlay
br_netfilter
EOF
```

### Load modules immediately
```bash
sudo modprobe overlay
sudo modprobe br_netfilter
```

### Verify loaded modules
```
lsmod | grep -E 'overlay|br_netfilter'
```
---

## Configure kernel parameters (sysctl)

Kubernetes requires proper packet forwarding and bridge traffic handling.

### Configure sysctl parameters

```bash
sudo tee /etc/sysctl.d/k8s.conf <<EOF
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                = 1
EOF
```

### Apply settings
```bash
sudo sysctl --system
```

### Verify settings
```bash
sysctl net.bridge.bridge-nf-call-iptables
sysctl net.ipv4.ip_forward
```
Expected value: 1

---

## Install container runtime (containerd)

Kubernetes requires a CRI-compatible container runtime.
This lab uses containerd.

### Install containerd

```bash
sudo apt-get update
sudo apt-get install -y containerd
```

### Configure containerd
```bash
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
```

### Enable systemd cgroup driver
Edit /etc/containerd/config.toml and ensure:

```bash
SystemdCgroup = true
```
You can do it automatically:

```bash
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
```

### Restart and enable containerd
```bash
sudo systemctl restart containerd
sudo systemctl enable containerd
```

### Verify containerd status
```
systemctl status containerd
```

---

## Verify system readiness
Before moving forward, ensure the following:
- swap is disabled
- required kernel modules are loaded
- sysctl parameters are applied
- containerd is running

This node is now ready for Kubernetes installation using kubeadm.

---
## Network ranges

| Network type | CIDR |
|--|--|
| Home LAN | 192.168.1.0/24 |
| Pod Network | 10.244.0.0/16 |

---
## Key design decisions

- Wired Ethernet only for cluster nodes
- Static addressing for predictability
- Dedicated Pod CIDR to avoid routing conflicts
- Single control plane (home lab scope)

This topology provides a simple, stable and production-like
foundation for Kubernetes experimentation.


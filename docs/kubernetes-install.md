# Kubernetes Installation (kubeadm - v1.34)

This section describes how to install Kubernetes components and initialize
the cluster using **kubeadm**.

At the end of this section, a **single-node control plane** will be up and running,
ready to accept worker nodes.

---

## Prerequisites

Before proceeding, ensure that:
- the operating system is prepared (see OS Preparation)
- swap is disabled
- containerd is running
- required kernel modules and sysctl parameters are applied

All commands below must be executed **as root or with sudo**.

---

## Install Kubernetes packages

### Add Kubernetes APT repository (v1.34)

Install required packages
```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg
```
Create keyrings directory (if not present)
```bash
sudo mkdir -p -m 755 /etc/apt/keyrings
```
Add CPG key
```bash
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.34/deb/Release.key \
| sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

Add Kubernetes APT repository
```bash
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] \
https://pkgs.k8s.io/core:/stable:/v1.34/deb/ /" \
| sudo tee /etc/apt/sources.list.d/kubernetes.list
```
Update package index
```bash
sudo apt-get update
```

---

### Install Kubernetes components (pinned version)

Select the Kubernetes version to install:
```bash
apt-cache madison kubeadm
```
Expected output:
```bash
kubeadm | 1.34.3-1.1 | https://pkgs.k8s.io/core:/stable:/v1.34/deb  Packages
kubeadm | 1.34.2-1.1 | https://pkgs.k8s.io/core:/stable:/v1.34/deb  Packages
kubeadm | 1.34.1-1.1 | https://pkgs.k8s.io/core:/stable:/v1.34/deb  Packages
kubeadm | 1.34.0-1.1 | https://pkgs.k8s.io/core:/stable:/v1.34/deb  Packages
```
Install kubeadm, kubelet and kubectl using the selected version
(example: 1.34.3-1.1):

```bash
sudo apt-get install -y \
kubelet=1.34.3-1.1 \
kubeadm=1.34.3-1.1 \
kubectl=1.34.3-1.1
```
Hold package versions

Prevent automatic upgrades to avoid version drift:

```bash
sudo apt-mark hold kubelet kubeadm kubectl
```

---

### Verify installation

```bash
kubeadm version
kubectl version --client
kubelet --version
```
Expected output:
- all components report v1.34.x

---
### Configure kubelet

Ensure kubelet uses the systemd cgroup driver, matching containerd.
This is already handled by containerd configuration, but kubelet must be restarted:

```bash
sudo systemctl daemon-reexec
sudo systemctl restart kubelet
```
At this point, kubelet will fail to start until the cluster is initialized.
This is expected behavior.

---

### Initialize the control plane

Run the following command on the control plane node only:
```bash
sudo kubeadm init \
--apiserver-advertise-address=192.168.1.53 \
--pod-network-cidr=10.244.0.0/16
```
The Pod CIDR must match the CNI configuration that will be installed later.

---
### Save the join command

At the end of the output, kubeadm prints a kubeadm join command.

⚠️ Save this command — it will be required to join worker nodes later.

---
### Configure kubectl access

Configure kubectl for the current user:

```bash
mkdir -p $HOME/.kube
sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Verify access:
```bash
kubectl get nodes
```
---
### Verify control plane status
Check core components:

```bash
kubectl get pods -n kube-system
```
Expected state:
- kube-apiserver running
- kube-controller-manager running
- kube-scheduler running
- etcd running

Some pods may be in Pending state until a CNI is installed — this is normal.










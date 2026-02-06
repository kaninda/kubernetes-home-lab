# Kubernetes Reset (kubeadm)

This section describes how to **cleanly reset a Kubernetes cluster**
installed with **kubeadm**.

A reset may be required in the following situations:
- failed or incomplete cluster initialization
- incorrect Pod CIDR or network configuration
- CNI installation issues
- certificate or kubeadm configuration errors
- reinstallation from scratch

The goal is to return the system to a **clean, pre-Kubernetes state**
without reinstalling the operating system.

---

## ⚠️ Important warning

This procedure:

- **removes all Kubernetes state**
- **deletes etcd data**
- **removes CNI configuration**
- **does not preserve workloads or cluster data**

Use it only when you intend to rebuild the cluster.

---

## Reset the control plane and worker nodes

The following steps **must be executed on all nodes**  
(control plane **and** workers).

### Run kubeadm reset

```bash
sudo kubeadm reset --force
```

This command:

- stops Kubernetes services
- removes static pod manifests
- deletes etcd data (on control plane)
- resets kubeadm state

---
## Remove Kubernetes configuration files

```bash
rm -rf $HOME/.kube
sudo rm -rf /etc/kubernetes
```
---

## Cleanup CNI configuration
### Remove CNI configuration and state:

```bash
sudo rm -rf /etc/cni/net.d
sudo rm -rf /var/lib/cni
```

### If Calico was used, also remove its state:

```bash
sudo rm -rf /var/lib/calico
```
---
## Cleanup iptables rules (recommended)

Kubernetes and CNI plugins modify iptables rules.
Reset them to avoid conflicts.

```bash
sudo iptables -F
sudo iptables -t nat -F
sudo iptables -t mangle -F
sudo iptables -X
```
---
## Restart container runtime and kubelet
```bash
sudo systemctl restart containerd
sudo systemctl restart kubelet
```
---
## Verify system state

### Ensure Kubernetes is no longer active:
```bash
kubectl get nodes
```

Expected:

- command fails or reports no configuration

Check that no Kubernetes ports are listening:

```bash
sudo ss -lntp | grep 6443
```

Expected:

- no output

---

## Optional: reboot the node

A reboot ensures:

- clean network state
- cleared virtual interfaces
- clean kernel routing tables

```bash
sudo reboot
```






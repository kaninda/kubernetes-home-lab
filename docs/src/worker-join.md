# Join Worker Nodes

This section describes how to join worker nodes to the Kubernetes cluster
using **kubeadm join**.

At the end of this section:
- all worker nodes will be part of the cluster
- nodes will be in `Ready` state
- Pod networking will work across nodes
---

## Prerequisites

Before joining worker nodes, ensure that:

- Kubernetes control plane is initialized
- Calico CNI is installed and running
- `kubeadm join` command is available
- Worker nodes completed **OS Preparation**
- Kubernetes packages are installed on worker nodes

Verify cluster status on the control plane:
```bash
kubectl get nodes
```
Expected output:

- control plane node is Ready

---
## Retrieve the join command

On the control plane node, retrieve (or regenerate) the join command:
```bash
sudo kubeadm token create --print-join-command
```
### Example output:
```bash
kubeadm join 192.168.1.53:6443 \
--token abcdef.0123456789abcdef \
--discovery-token-ca-cert-hash sha256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```
⚠️ This command is unique and time-limited.
---
## Join worker nodes
### Execute on each worker node

Run the kubeadm join command on each worker node only:
```bash
sudo kubeadm join 192.168.1.53:6443 \
--token abcdef.0123456789abcdef \
--discovery-token-ca-cert-hash sha256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```
### This command:

- registers the node with the API server
- installs required certificates
- starts kubelet on the worker node

---
### Verify node registration

On the control plane, verify node status:
```bash
kubectl get nodes
```
Expected output:

- control plane node: Ready
- worker nodes: Ready

### Example:
```bash
NAME        STATUS   ROLES           AGE     VERSION
cp-1        Ready    control-plane   20m     v1.34.x
worker-1    Ready    <none>           2m     v1.34.x
worker-2    Ready    <none>           2m     v1.34.x
```
---
## Verify Pod scheduling across nodes

### Deploy a test Pod:
```bash
kubectl run nginx-test --image=nginx --restart=Never
```

### Check where the Pod is scheduled:
```bash
kubectl get pod nginx-test -o wide
```

### Expected:

- Pod is Running
- Pod is scheduled on a worker node
- Pod has an IP from the Pod CIDR range
---

## Cleanup test Pod
```bash
kubectl delete pod nginx-test
```
---

## Common issues

### Node stays in NotReady

Check CNI status:

```bash
kubectl get pods -n kube-system
```

Ensure:

- calico-node is running on the worker node
- no CrashLoopBackOff
---

## Join token expired

### Generate a new token:

```bash
sudo kubeadm token create --print-join-command
```

### Result

At this stage:

- all nodes are successfully joined
- Pod networking works across nodes
- the Kubernetes cluster is fully operational





# Cluster Validation

This section validates that the Kubernetes cluster is **healthy, functional
and ready for workloads**.

The checks below are intentionally simple and reproducible.  
They focus on **core Kubernetes components**, **networking**, and **basic scheduling**.

---

## Node status

Verify that all nodes are registered and ready:

```bash
kubectl get nodes -o wide
```
### Expected:

- all nodes are in Ready state
- control plane has the role control-plane
- worker nodes have no role
- correct internal IPs are reported
---
## System pods status

### Check core system components:

```bash
kubectl get pods -n kube-system
```
### Expected:

- kube-apiserver: Running
- kube-controller-manager: Running
- kube-scheduler: Running
- etcd: Running
- calico-node: Running on all nodes
- calico-kube-controllers: Running

**No pods should be in CrashLoopBackOff or Pending.**

---
## Component health

### Verify control plane component health:

```bash
kubectl get componentstatuses
```

Expected:

- all components report Healthy

**Note**: This command is deprecated but still useful for quick validation.

---

## Cluster events

Inspect recent cluster events:

```bash
kubectl get events -A --sort-by=.metadata.creationTimestamp
```

### Expected:

- no recurring errors
- no certificate or networking failures

Warnings may appear briefly during startup but should not persist.

---

## DNS resolution

### Validate CoreDNS functionality:

```bash
kubectl run dns-test \
--image=busybox \
--restart=Never \
--rm -it -- \
nslookup kubernetes.default
```
### Expected:

- DNS resolution succeeds
- service IP is returned

---
## Pod-to-Pod networking

### Deploy two test pods on different nodes:

```bash
kubectl run pod-a --image=busybox --restart=Never -- sleep 3600
kubectl run pod-b --image=busybox --restart=Never -- sleep 3600
```

### Check pod placement and IPs:
```bash
kubectl get pods -o wide
```

### Verify connectivity:
```bash
kubectl exec pod-a -- ping -c 3 <POD-B-IP>
```

### Expected:

- ICMP traffic succeeds
- no packet loss

---

## Cleanup test pods
```bash
kubectl delete pod pod-a pod-b
```
---

## API server access

### Verify API access and permissions:

```bash
kubectl auth can-i get pods --all-namespaces
```

### Expected:

- yes

---
## Summary

### At this stage:

- All nodes are Ready
- Core control plane components are healthy
- Calico networking is functional
- DNS resolution works
- Pod-to-Pod communication is verified

**The Kubernetes cluster is fully operational and ready for workloads.**
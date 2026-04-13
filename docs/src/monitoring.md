# Monitoring Stack

This section describes how the **kube-prometheus-stack** was installed and configured
to monitor the Kubernetes Home Lab cluster.

The stack provides full observability across all nodes and workloads:
metrics collection, visualization, and alerting — all exposed securely
via Cloudflare Tunnel and Gateway API.

---

## Architecture

![Monitoring Architecture](diagrams/monitoring.svg){ width="100%" }

## Components

| Component | Pod | Role |
|---|---|---|
| `prometheus` | `prometheus-kube-prometheus-stack-prometheus-0` | Metrics collection and storage |
| `grafana` | `kube-prometheus-stack-grafana-xxxx` | Visualization dashboards |
| `alertmanager` | `alertmanager-kube-prometheus-stack-alertmanager-0` | Alert routing |
| `prometheus-operator` | `kube-prometheus-stack-operator-xxxx` | Manages Prometheus/Alertmanager CRDs |
| `kube-state-metrics` | `kube-prometheus-stack-kube-state-metrics-xxxx` | Kubernetes object metrics |
| `node-exporter` | `kube-prometheus-stack-prometheus-node-exporter-xxxx` ×3 | Node-level metrics (1 per node) |

---

## Installation

The stack is installed via **Helm** using the `kube-prometheus-stack` chart.

### Add the Helm repository

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

### Create the namespace

```bash
kubectl create namespace monitoring
```

### Install the stack

```bash
helm install kube-prometheus-stack prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --set prometheus.prometheusSpec.retention=7d \
  --set alertmanager.enabled=true \
  --set grafana.enabled=true
```

!!! info
**Chart version** : `kube-prometheus-stack-83.4.0`
**App version** : `v0.90.1` (prometheus-operator)

### Verify the installation

```bash
kubectl get pods -n monitoring
```

Expected — all pods Running:

```
alertmanager-kube-prometheus-stack-alertmanager-0           2/2   Running
kube-prometheus-stack-grafana-xxxx                          3/3   Running
kube-prometheus-stack-kube-state-metrics-xxxx               1/1   Running
kube-prometheus-stack-operator-xxxx                         1/1   Running
kube-prometheus-stack-prometheus-node-exporter-xxxx         1/1   Running  (×3)
prometheus-kube-prometheus-stack-prometheus-0               2/2   Running
```

---

## Bare-Metal Fixes

On a bare-metal cluster, several components listen on `127.0.0.1` (loopback)
by default and are not reachable by Prometheus.

Verify which targets are down:

```promql
up == 0
```

The following 6 targets were down and required manual fixes.

---

### kube-proxy

Edit the ConfigMap (applied to all 3 nodes via DaemonSet):

```bash
kubectl edit configmap kube-proxy -n kube-system
# metricsBindAddress: "" → metricsBindAddress: "0.0.0.0:10249"

kubectl rollout restart daemonset kube-proxy -n kube-system
```

### kube-scheduler

Edit the static pod manifest on `cp-1`:

```bash
sudo vi /etc/kubernetes/manifests/kube-scheduler.yaml
# --bind-address=127.0.0.1 → --bind-address=0.0.0.0
```

---

### kube-controller-manager

```bash
sudo vi /etc/kubernetes/manifests/kube-controller-manager.yaml
# --bind-address=127.0.0.1 → --bind-address=0.0.0.0
```

---

### etcd

```bash
sudo vi /etc/kubernetes/manifests/etcd.yaml
# --listen-metrics-urls=http://127.0.0.1:2381 → --listen-metrics-urls=http://0.0.0.0:2381
```

!!! note
Static pod manifests are automatically reloaded by kubelet — no manual restart needed.
For kube-proxy, a DaemonSet rollout restart is required to apply the ConfigMap change.

### Verify all targets are up

```promql
up == 0
```

Expected result: **Empty query result** — all targets are UP. ✅

---

## Exposure via Gateway API

Three HTTPRoutes are deployed in the `monitoring` namespace,
all pointing to the shared `edge-gateway` in `nginx-gateway`.

```bash
kubectl get httproute -n monitoring
```

```
NAME           HOSTNAMES                        AGE
alertmanager   ["alertmanager.kanismile.com"]   ...
grafana        ["grafana.kanismile.com"]         ...
prometheus     ["prometheus.kanismile.com"]      ...
```

### HTTPRoute example (Grafana)

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: grafana
  namespace: monitoring
spec:
  parentRefs:
    - name: edge-gateway
      namespace: nginx-gateway
  hostnames:
    - grafana.kanismile.com
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /
      backendRefs:
        - name: kube-prometheus-stack-grafana
          port: 80
```

The same pattern applies for Prometheus (port `9090`) and Alertmanager (port `9093`).

Apply all routes:

```bash
kubectl apply -f httproute/httproute-prometheus.yaml
kubectl apply -f httproute/httproute-grafana.yaml
kubectl apply -f httproute/httproute-alertmanager.yaml
```

---

## Access

| Service | URL | Auth |
|---|---|---|
| Grafana | `https://grafana.kanismile.com` | `admin` / `prom-operator` |
| Prometheus | `https://prometheus.kanismile.com` | ❌ No auth |
| Alertmanager | `https://alertmanager.kanismile.com` | ❌ No auth |

!!! warning
Prometheus and Alertmanager are currently accessible without authentication.
It is recommended to add a **Cloudflare Access** policy in front of these
two URLs to restrict access.

---

## Design Decisions

- **Helm install** — `kube-prometheus-stack` bundles all components in a single release
- **Retention set to 7 days** — appropriate for a home lab, avoids excessive disk usage
- **Bare-metal bind address fixes** — required for full cluster visibility on kubeadm setups
- **ClusterIP only** — all services remain internal, exposure handled by Gateway API

---

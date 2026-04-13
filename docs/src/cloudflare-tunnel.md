# Cloudflare Tunnel

This section describes how **Cloudflare Tunnel** is used to expose cluster services
to the internet without any public IP, open port, or LoadBalancer.

All external traffic enters the cluster through a single secure outbound connection
established by the `cloudflared` agent running inside the cluster.

---

## Architecture

![Cloudflare Tunnel Architecture](diagrams/cloudflare-tunnel.svg){ width="100%" }

- **TLS is terminated at the Cloudflare edge** — traffic inside the cluster is plain HTTP
- **No inbound ports** are opened on the cluster nodes
- **One tunnel** handles all services via hostname-based routing

---

## How It Works

The `cloudflared` agent runs inside the cluster and maintains a persistent
**outbound connection** to the Cloudflare edge network.

When a user requests `https://argocd.kanismile.com`:

1. Cloudflare resolves the domain and routes the request through the tunnel
2. `cloudflared` receives the request inside the cluster
3. It forwards the request to `edge-gateway-nginx.nginx-gateway.svc.cluster.local:80`
4. NGINX Gateway Fabric matches the hostname via the corresponding HTTPRoute
5. The request is forwarded to the target service

---

## Tunnel Configuration

The tunnel was created via the **Cloudflare Zero Trust dashboard** and configured
with one public hostname per service, all pointing to the same internal service:

| Hostname | Internal Service |
|---|---|
| `argocd.kanismile.com` | `http://edge-gateway-nginx.nginx-gateway.svc.cluster.local:80` |
| `grafana.kanismile.com` | `http://edge-gateway-nginx.nginx-gateway.svc.cluster.local:80` |
| `prometheus.kanismile.com` | `http://edge-gateway-nginx.nginx-gateway.svc.cluster.local:80` |
| `alertmanager.kanismile.com` | `http://edge-gateway-nginx.nginx-gateway.svc.cluster.local:80` |

!!! info
All hostnames point to the same internal URL.
Hostname-based routing is handled by the HTTPRoutes at the Gateway layer —
not by the tunnel itself.

---

## Deployment

The `cloudflared` agent runs as a **Deployment** in the `cloudflare` namespace.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cloudflared
  namespace: cloudflare
spec:
  replicas: 1
  selector:
    matchLabels:
      app: cloudflared
  template:
    metadata:
      labels:
        app: cloudflared
    spec:
      containers:
        - name: cloudflared
          image: cloudflare/cloudflared:latest
          args:
            - tunnel
            - run
          env:
            - name: TUNNEL_TOKEN
              valueFrom:
                secretKeyRef:
                  name: cloudflared-token
                  key: TUNNEL_TOKEN
          resources:
            limits:
              memory: "128Mi"
              cpu: "100m"
```

Apply it with:

```bash
kubectl create namespace cloudflare
kubectl apply -f cloudflare/cloudflared.yaml
```

---

## Tunnel Token Secret

The tunnel token is stored as a Kubernetes Secret and injected via environment variable.

Create the secret from the token provided by the Cloudflare Zero Trust dashboard:

```bash
kubectl create secret generic cloudflared-token \
  --from-literal=TUNNEL_TOKEN=<your-tunnel-token> \
  --namespace cloudflare
```

!!! warning
Never commit the tunnel token to Git.
The secret must be created manually on the cluster before applying the Deployment.

---

## Verify the Deployment

```bash
kubectl get pods -n cloudflare
```

Expected output:

```
NAME                           READY   STATUS    RESTARTS   AGE
cloudflared-xxxx               1/1     Running   0          ...
```

Check the tunnel is connected from the Cloudflare Zero Trust dashboard:

**Zero Trust → Networks → Tunnels → k8s-homelab → Status: Healthy**

---

## Design Decisions

- **Single tunnel for all services** — one `cloudflared` instance handles all hostnames
- **All routes point to the Gateway** — routing logic stays in Kubernetes (HTTPRoutes), not in Cloudflare
- **Token via Secret** — the tunnel token is never stored in Git
- **No public IP required** — the cluster nodes have no inbound exposure

!!! tip
To add a new service, simply create a new HTTPRoute in the cluster
and add the corresponding hostname in the Cloudflare Zero Trust dashboard
pointing to `http://edge-gateway-nginx.nginx-gateway.svc.cluster.local:80`.

---

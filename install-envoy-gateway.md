# Install Envoy Gateway on Your Cluster

A step-by-step guide to replacing Ingress NGINX with Envoy Gateway (Gateway API) on a bare-metal / kubeadm cluster that uses NodePort and has no cloud load balancer.

---

## Step 1: Check requirements

You need Kubernetes 1.25 or later, `kubectl`, and Helm.

```bash
kubectl version
helm version
```

If Helm is missing, install it first (on Ubuntu: `sudo snap install helm --classic`).

---

## Step 2: Install Envoy Gateway with Helm

The Helm chart is published at `oci://docker.io/envoyproxy/gateway-helm`. The install also applies the Gateway API CRDs.

```bash
helm install eg oci://docker.io/envoyproxy/gateway-helm \
  --version v1.6.7 \
  -n envoy-gateway-system --create-namespace

helm install eg oci://docker.io/envoyproxy/gateway-helm --version v0.0.0-latest -n envoy-gateway-system --create-namespace

```

> A newer version may exist. Check the current install docs: https://gateway.envoyproxy.io/latest/install/install-helm/

---

## Step 3: Wait until it is ready

```bash
kubectl wait --timeout=5m -n envoy-gateway-system \
  deployment/envoy-gateway --for=condition=Available
```

---

## Step 4: Make the gateway use a NodePort

Without a cloud load balancer, a `LoadBalancer` service stays `<pending>` forever. This config tells Envoy Gateway to create a NodePort service instead.

```bash
kubectl apply -f - <<'EOF'
apiVersion: gateway.envoyproxy.io/v1alpha1
kind: EnvoyProxy
metadata:
  name: custom-proxy
  namespace: envoy-gateway-system
spec:
  provider:
    type: Kubernetes
    kubernetes:
      envoyService:
        type: NodePort
EOF
```

---

## Step 5: Create the GatewayClass and Gateway

The GatewayClass says "Envoy Gateway handles this". The Gateway opens port 80.

```bash
kubectl apply -f - <<'EOF'
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: eg
spec:
  controllerName: gateway.envoyproxy.io/gatewayclass-controller
  parametersRef:
    group: gateway.envoyproxy.io
    kind: EnvoyProxy
    name: custom-proxy
    namespace: envoy-gateway-system
---
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: eg-gateway
  namespace: default
spec:
  gatewayClassName: eg
  listeners:
  - name: http
    protocol: HTTP
    port: 80
EOF
```

---

## Step 6: Route traffic to nginx-service

The HTTPRoute replaces an Ingress. It sends requests for `www.example.com` to the existing `nginx-service` on port 8080.

```bash
kubectl apply -f - <<'EOF'
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: nginx-route
  namespace: default
spec:
  parentRefs:
  - name: eg-gateway
  hostnames:
  - "www.example.com"
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /
    backendRefs:
    - name: nginx-service
      port: 8080
EOF
```

---

## Step 7: Check that everything is healthy

```bash
kubectl get gatewayclass
kubectl get gateway
kubectl get httproute
```

`GatewayClass` and `Gateway` should show `True` / `Accepted` or `Programmed`. If a resource is not ready, run `kubectl describe` on it and read the Conditions at the bottom.

---

## Step 8: Find the NodePort and test

```bash
kubectl get svc -n envoy-gateway-system \
  -l gateway.envoyproxy.io/owning-gateway-name=eg-gateway
```

You will see something like `80:31234/TCP`. The number after the colon is your NodePort. Then test:

```bash
curl -H "Host: www.example.com" http://<node-ip>:31234/
```

You should see the nginx welcome page. The `Host` header is required because the route matches on the hostname. To use the name in a browser, add this line to `/etc/hosts`:

```
<node-ip> www.example.com
```

---

## How the pieces fit

```
Client -> NodePort -> Envoy proxy pod -> HTTPRoute rules -> nginx-service:8080 -> nginx pods
          (Step 4)    (Step 5 Gateway)    (Step 6)
```

---

## Troubleshooting

| Problem | What to check |
|---|---|
| Gateway not `Programmed` | `kubectl describe gateway eg-gateway` |
| `curl` returns 404 | The `Host` header does not match `hostnames` in the HTTPRoute |
| `curl` returns 503 | `kubectl get endpoints nginx-service` is empty, so the selector matches no pods |
| No Envoy pod | `kubectl get pods -n envoy-gateway-system` |

---

## Clean up (optional)

```bash
kubectl delete httproute nginx-route
kubectl delete gateway eg-gateway
kubectl delete gatewayclass eg
kubectl delete envoyproxy custom-proxy -n envoy-gateway-system
helm uninstall eg -n envoy-gateway-system
```

# Deploy Istio and Application Layer Policy


### 1. Deploy Istio

```
curl -L https://git.io/getLatestIstio | ISTIO_VERSION=1.0.7 sh - 
cd $(ls -d istio-*)
kubectl apply -f install/kubernetes/helm/istio/templates/crds.yaml
kubectl apply -f install/kubernetes/istio-demo-auth.yaml
watch kubectl get pods -n istio-system -o wide
```

Proceed further after all Istio control plane pods have started - it could take a few minutes

### 2. Deploy ALP

Enable Dikastes sidecar injection alongside istio-proxy, and make istio-proxy query it for Policy authorization decisions.
```
kubectl apply -f https://docs.tigera.io/v2.4/getting-started/kubernetes/installation/manifests/app-layer-policy/istio-inject-configmap.yaml
kubectl apply -f https://docs.tigera.io/v2.4/getting-started/kubernetes/installation/manifests/app-layer-policy/istio-app-layer-policy.yaml
```

### 3. Convert Istio Addons from service type ClusterIP into LoadBalancer

In the Spec: section, change the service from type: ClusterIP to type: LoadBalancer for each of the services below.

```
# OPTION 1: Manually edit each service:
kubectl edit svc -n istio-system jaeger-query
kubectl edit svc -n istio-system servicegraph
kubectl edit svc -n istio-system grafana

# # OPTION 2: You may also patch the services, by doing the following:
## Back up your current services:
kubectl get svc -n istio-system jaeger-query -o yaml > svc-jaeger-query.yaml
kubectl get svc -n istio-system servicegraph -o yaml > svc-servicegraph.yaml
kubectl get svc -n istio-system grafana -o yaml > svc-grafana.yaml

## Then patch the current services in-place:
kubectl patch svc jaeger-query -n istio-system -p '{"spec": {"ports": [{"port": 16686,"targetPort": 16686,"name": "query-http"}],"type": "LoadBalancer"}}'
kubectl patch svc servicegraph -n istio-system -p '{"spec": {"ports": [{"port": 8088,"targetPort": 8088,"name": "servicegraph"}],"type": "LoadBalancer"}}'
kubectl patch svc grafana -n istio-system -p '{"spec": {"ports": [{"port": 3000,"targetPort": 3000,"name": "http"}],"type": "LoadBalancer"}}'
```
It will take a couple of minutes for the LoadBalancer IP to be assigned by Azure. Repeat the commands below until they each return a Load Balancer IP.

### 4. Get LoadBalancer IP's of Addons and browse to them
```
kubectl get svc -n istio-system grafana -o 'jsonpath={.status.loadBalancer.ingress[*].ip}'
kubectl get svc -n istio-system jaeger-query -o 'jsonpath={.status.loadBalancer.ingress[*].ip}'
kubectl get svc -n istio-system servicegraph -o 'jsonpath={.status.loadBalancer.ingress[*].ip}'
```

The corresponding URLs to browse to are http://GrafanaIP:3000/dashboard/db/istio-mesh-dashboard http://jaeger-queryIP:16686/ and http://ServiceGraphIP:8088/force/forcegraph.html



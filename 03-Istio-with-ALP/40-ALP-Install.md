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

### 3. Convert Istio Addons from servicetype ClusterVIP into LoadBalancer

Change from serviceType: ClusterVIP to serviceType: LoadBalancer for each of the services below.

```
kubectl edit svc -n istio-system jaeger-query
kubectl edit svc -n istio-system servicegraph
kubectl edit svc -n istio-system grafana
```


### 4. Get LoadBalancer IP's of Addons and browse to them
```
kubectl get svc -n istio-system grafana -o 'jsonpath={.status.loadBalancer.ingress[*].ip}'
kubectl get svc -n istio-system jaeger-query -o 'jsonpath={.status.loadBalancer.ingress[*].ip}'
kubectl get svc -n istio-system servicegraph -o 'jsonpath={.status.loadBalancer.ingress[*].ip}'
```

The corresponding URLs to browse to are http://GrafanaIP:3000/ http://jaeger-queryIP:16686/ and http://ServiceGraphIP:8088/force/forcegraph.html



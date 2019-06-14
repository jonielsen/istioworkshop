# Verify Istio Add-On services are running

```
kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=grafana -o jsonpath='{.items[0].metadata.name}') 3000:3000


kubectl -n istio-system port-forward $(kubectl -n istio-system get pod -l app=prometheus -o jsonpath='{.items[0].metadata.name}') 9090:9090


kubectl port-forward -n istio-system $(kubectl get pod -n istio-system -l app=jaeger -o jsonpath='{.items[0].metadata.name}') 16686:16686


kubectl port-forward -n istio-system $(kubectl get pod -n istio-system -l app=kiali -o jsonpath='{.items[0].metadata.name}') 20001:20001
```

# Add the istio-injection label to the default namespace. 

kubectl label namespace default istio-injection=enabled


# Verify and record the Istio Ingress service public IP address

kubectl get service istio-ingressgateway --namespace istio-system -o jsonpath='{.status.loadBalancer.ingress[0].ip}'


# Install the first application for this workshop

Clone down the following repo: https://github.com/jonielsen/istioworkshop.git

Switch to the Istio directory

kubectl apply -f my-websites.yaml and verify the applications are running


# Configure the Gateway and Virtual Service to get traffic from outside of the cluster to these services

kubectl apply -f website-routing.yaml

Use the ingress svc public IP and verify you can hit this service in a browser

This configuration will target only version1 one of the application.


# Configure the Virtual Service to now send 10 percent of traffic to version2 and 90 percent to version1. 

kubectl apply -f website-routing-v2.yaml

Test this in the browser and you should occasionally see traffic hit v2. 







 

 



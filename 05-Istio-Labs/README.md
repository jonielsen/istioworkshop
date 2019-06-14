## Istio Labs - 1 hour

```

## Add the istio-injection label to the default namespace.

kubectl label namespace default istio-injection=enabled


## Verify and record the Istio Ingress service public IP address

kubectl get service istio-ingressgateway --namespace istio-system -o jsonpath='{.status.loadBalancer.ingress[0].ip}'


## Install the first application for this workshop

Clone down the following repo: https://github.com/jonielsen/istioworkshop.git

Switch to the Istio directory

kubectl apply -f my-websites.yaml and verify the application is running

kubectl get pods 


## The istio-proxy container has automatically been injected by Istio to manage the network traffic to and from your components, as shown in the following example output:

## Configure the Gateway and Virtual Service to get traffic from outside of the cluster to these services

kubectl apply -f website-routing.yaml

Use the ingress svc public IP and verify you can hit this service in a browser

This configuration will target only version1 one of the application.


# Configure the Virtual Service to now send 10 percent of traffic to version2 and 90 percent to version1.

kubectl apply -f website-routing-v2.yaml

Test this in the browser and you should occasionally see traffic hit v2.


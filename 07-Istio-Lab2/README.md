# Istio Lab 2 - 20 minutes

Remove the virtual service, gateway, and destination rules from the previous application

```
kubectl get gateway
kubectl get virtualservice
kubectl get destinationrules
```
Now use the output from the above commands to delete the above resources

```
kubectl delete gateway name from above
kubectl delete virtualservice name from above
kubectl delete destinationrules name from above
```

# Install the second application for this workshop

```
cd istioworkshop/07-Istio-Lab2/
```

Deploy the applications

```
kubectl apply -f deployapp.yaml
```

Verify the application is running

```
kubectl get pods 
```

Deploy the destionrules and virtual service

```
kubectl apply -f deployapp.yaml


# Configure the Gateway and Virtual Service to get traffic to the /productpage site 

```
kubectl apply -f gateway.yaml 
```

```
curl -s http://INGRESS_IP_PORT/productpage | grep -o "<title>.*</title>"
```


# Configure the destionation rules

kubectl apply -f destination-rules.yaml



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

# Deploy the Destionrules and Virtual Service

```
kubectl apply -f deployapp.yaml
```

# Configure the Gateway and Virtual Service to get traffic to the /productpage site 

```
kubectl apply -f gateway.yaml 
```

Verify you can hit the product page

```
curl -s http://INGRESS_IP_PORT/productpage | grep -o "<title>.*</title>"
```

# Configure the destionation rules and a new Virtual Service to route to v1

```
kubectl apply -f destination-rules.yaml

kubectl apply -f virtual-service-1.yaml 
```


kubectl get destinationrules -o yaml

You have configured Istio to route to the v1 version of the Bookinfo microservices, most importantly the reviews service version 1.

Notice that the reviews part of the page displays with no rating stars, no matter how many times you refresh. This is because you configured Istio to route all traffic for the reviews service to the version reviews:v1 and this version of the service does not access the star ratings service.


Now let's route to a service based on the user logging into the page

```
kubectl apply -f virtual-service-2.yaml
```

On the /productpage of the Bookinfo app, log in as user jason.

Refresh the browser. What do you see? The star ratings appear next to each review.

Log in as another user (pick any name you wish).

Refresh the browser. Now the stars are gone. This is because traffic is routed to reviews:v1 for all users except Jason.





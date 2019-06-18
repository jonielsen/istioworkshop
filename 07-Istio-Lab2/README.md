#Istio Labs - 20 minutes

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
```



The following example output shows there are three different versions of the web application running in the cluster. Each of the pods has two containers. One of these containers is the web app, and the other is the envoy-proxy:

```
jordan@Azure:~$ kubectl get pods
NAME                          READY   STATUS    RESTARTS   AGE
web-v1-799fc6769f-xlqq2       2/2     Running   0          7h16m
web-v2-76c8d8bbfd-6qsn6       2/2     Running   0          7h16m
web-v3-659f996db9-n9lgs       2/2     Running   0          7h16m
```

To see information about the pod, use the kubectl describe pod. Replace the pod name with the name of a pod in your own AKS cluster from the previous output:

```
kubectl describe pod web-v1-799fc6769f-xlqq2
```

The istio-proxy container has automatically been injected by Istio to manage the network traffic to and from the web application, as shown in the following example output:

```
Containers:
  website-version-2:
    Container ID:   docker://07c54009578de7bdc383cfa11d991e25b952799bab13c07fdd18446fc4006e44
    Image:          kublr/kublr-tutorial-images:v2
    Image ID:       docker-pullable://kublr/kublr-tutorial-images@sha256:cb0bcd7336e26ba4ad35eee55571566849b292cf833a124e45727c04e8c1625d
  istio-proxy:
    Container ID:  docker://32c49b2304b11bce38622525899e9cf1ef1874debab45188c2547cc58460b281
    Image:         docker.io/istio/proxyv2:1.1.3
    Image ID:      docker-pullable://istio/proxyv2@sha256:b682918f2f8fcca14b3a61bbd58f4118311eebc20799f24b72ceddc5cd749306
    Port:          15090/TCP
```

# Configure the Gateway and Virtual Service to get traffic from outside of the cluster to the web applications

```
kubectl apply -f website-routing.yaml
```

Use the ingress svc public IP and verify you can hit this service in a browser

This configuration will target only version1 one of the application.

# Configure the Virtual Service to now send 10 percent of traffic to version2 and 90 percent to version1.

kubectl apply -f website-routing-v2.yaml

Test this in the browser and you should occasionally see traffic hit v2.


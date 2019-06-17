
# Defense in Depth Demo with Calico ALP for Istio and Calico NP for Kubernetes

### 1. Deploy YAOBank microservices

YAOBank is a Banking demo application with 3 microservices (customer, summary and database)
```
kubectl apply -n yaobank -f 10-yaobank.yaml
kubectl get pods -n yaobank
kubectl get serviceaccount -n yaobank
kubectl get secret -n yaobank
```

You should see the pods running, and the service accounts and secrets created.


### 2. Access Customer front end application to see your bank balance

Get Istio Ingressgateway LoadBalancer IP
```
kubectl get svc istio-ingressgateway -n istio-system  -o 'jsonpath={.status.loadBalancer.ingress[*].ip}'
```

Browse to http://IngressgatewayIP/ to see the customers bank balance.

### 3. Hack into the customer pod, and try to attack the database directly

```
# Determine customer pod name
CUSTOMER_POD=$(kubectl get pod -l app=customer -n yaobank -o jsonpath='{.items[0].metadata.name}')

# Execute bash command in customer container
kubectl exec -ti $CUSTOMER_POD -n yaobank -c customer bash
```

Now curl the database directly from the customer pod bypassing the summary microservice
```
curl -v http://database:2379/v2/keys?recursive=true | python -m json.tool
exit
```

Obviously not good, you can see everyone's bank balance


### 4. Launch an Attack pod with access to the X.509 certs provided to the istio-proxy (envoy) sidecar.

```
kubectl apply -n yaobank -f 20-attack-pod.yaml
watch kubectl get pods -n yaobank
```

Launch attack against the database, this time from the attack pod with access to the X.509 (SPIFFE) certs.
In Istio 1.0, the certificates are Kubernetes secrets mounted into istio-proxy/Envoy. Istio 1.1 uses the Envoy
SDS api to make the certs available to istio-proxy/Envoy.
```
# Determine attack pod name
ATTACK_POD=$(kubectl get pod -l app=attack -n yaobank -o jsonpath='{.items[0].metadata.name}')

# Execute bash command in attack container
kubectl exec -ti $ATTACK_POD -n yaobank bash
```

Lets change the balance in the account.
```
curl -k -v https://database:2379/v2/keys/accounts/519940/balance -d value="9999.01"  -XPUT --key /etc/certs/key.pem --cert /etc/certs/cert-chain.pem
exit
```

Now browse to the customer front end, what do you see as the balance?


### 5. Providing Defense in Depth with Calico ALP for Istio and Envoy together with Calico Network Policy for Kubernetes

Lets create some policies for defense in depth.
```
kubectl apply -f 30-defend-policy.yaml
```

Notice that these policies have been created in a higher precedence (Line of Business) tier called lob-banking. So these take precedence over developer policies in the default tier.

Now launch the same attack described in steps 3 and 4 above. What is the result?


# TSEE 2.4 - Manual Install


### 1. Update AKS azure-cni deployment from Bridge mode to Transparent Mode.

Apply manifest to change CNI configuration and have kured reboot nodes

```
kubectl apply -f https://raw.githubusercontent.com/jonielsen/istioworkshop/master/03-TigeraSecure-Install/bridge-to-transparent.yaml
```

Watch nodes and pods to ensure that all nodes have rebooted

```
watch kubectl get pods --all-namespaces -o wide 
kubectl get nodes -o wide
kubectl describe node <node-name> |grep reboot
```

Proceed further after every node has rebooted. This might take a few minutes for your cluster, depending on it's size.


### 2. Deploy TSEE Core

```
wget https://github.com/jonielsen/istioworkshop/blob/master/03-TigeraSecure-Install/tigera-secure-ee-core-aks-v2.4.1-0.tgz
wget https://github.com/jonielsen/istioworkshop/blob/master/03-TigeraSecure-Install/tigera-secure-ee-aks-v2.4.1-0.tgz
```

Upload these Helm charts along with the Tigera pull secret (aks-tsee-workshop-auth.json) and License (tigera-aks-workshop-license.yaml) to your Azure Cloud Shell (if you're deploying from the Cloud Shell)


Deploy TSEE Core

```
helm install ./tigera-secure-ee-aks-v2.4.1-0.tgz --set-file imagePullSecrets.cnx-pull-secret=./aks-tsee-workshop-auth.json

watch kubectl get pods -n kube-system -o wide
```

Proceed after all the calico-node pods are ready (1/1)

Verify that cnx-apiserver rolled out

```
kubectl rollout status -n kube-system deployment/cnx-apiserver
```

Apply the TSEE License and failsafe policies
```
kubectl apply -f ./tigera-aks-workshop-license.yaml
kubectl apply -f https://docs.tigera.io/v2.4/getting-started/kubernetes/installation/hosted/cnx/1.7/cnx-policy.yaml
```



### 3. Deploy TSEE Addons

Deploy TSEE Addons

```
helm install ./tigera-secure-ee-aks-v2.4.1-0.tgz  --namespace calico-monitoring --set createCustomResources=false --set-file imagePullSecrets.cnx-pull-secret=../aks-workshop/aks-tsee-workshop-auth.json
watch kubectl get pods --all-namespaces -o wide
```

This will take a few minutes. Proceed further after elastic-tsee-installer shows a status of "Completed".

### 4. Access UI

Create Service Account for UI Access
```
export USER=tigera-admin
export NAMESPACE=kube-system
kubectl create serviceaccount -n $NAMESPACE $USER
kubectl create clusterrolebinding admin-tigera  --clusterrole=tigera-ui-user --serviceaccount=kube-system:tigera-admin
kubectl create clusterrolebinding tigera-network-admin --clusterrole=network-admin --serviceaccount=kube-system:tigera-admin
kubectl get secret -n $NAMESPACE -o jsonpath='{.data.token}' $(kubectl -n $NAMESPACE get secret | grep $USER | awk '{print $1}') | base64 --decode
```

Find LoadBalancer IP for UI
```
kubectl get svc -n calico-monitoring cnx-manager -o 'jsonpath={.status.loadBalancer.ingress[*].ip}'
```

Browse to https://IPAddressofUI-Loadbalancer:9443/ and log in with the token above

Set Kibana Defaults.

```
kubectl get svc -n calico-monitoring tigera-kibana -o 'jsonpath={.status.loadBalancer.ingress[*].ip}'
```

Make a note of that IP - this is your Kibana IP. Browse to http://KibanaIP:5601/ and:
- Click on Dashboard
- Click on TSEE Flow Logs
- Click on the Star at the top right (to make Flow logs the default index)



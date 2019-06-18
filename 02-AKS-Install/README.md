# AKS Install - 20-30 minutes

```
SP_PASSWORD=mySecurePassword \
RESOURCE_GROUP_NAME=myResourceGroup-NP \
CLUSTER_NAME=myAKSCluster \
LOCATION=westus2
```

# Create resource group

```
az group create --name $RESOURCE_GROUP_NAME --location $LOCATION
```

# Create a virtual network and subnet

```
az network vnet create \
    --resource-group $RESOURCE_GROUP_NAME \
    --name myVnet \
    --address-prefixes 10.0.0.0/8 \
    --subnet-name myAKSSubnet \
    --subnet-prefix 10.240.0.0/16
```

# Create a service principal and read in the application ID

```
SP_ID=$(az ad sp create-for-rbac --password $SP_PASSWORD --skip-assignment --query [appId] -o tsv)
```

# Wait 15 seconds to make sure that service principal has propagated

```
echo "Waiting for service principal to propagate..."
sleep 15
```

# Get the virtual network resource ID

```
VNET_ID=$(az network vnet show --resource-group $RESOURCE_GROUP_NAME --name myVnet --query id -o tsv)
```

# Assign the service principal Contributor permissions to the virtual network resource

```
az role assignment create --assignee $SP_ID --scope $VNET_ID --role Contributor
```

# Get the virtual network subnet resource ID

```
SUBNET_ID=$(az network vnet subnet show --resource-group $RESOURCE_GROUP_NAME --vnet-name myVnet --name myAKSSubnet --query id -o tsv)
```

Create the AKS cluster and specify the virtual network and service principal information.

Deploy cluster without the --network-policy calico option.

Min cluster size of 3 nodes is suggested due to various Istio components and Elastic sizing.

```
az aks create \
    --resource-group $RESOURCE_GROUP_NAME \
    --name $CLUSTER_NAME \
    --node-count 3 \
    --generate-ssh-keys \
    --network-plugin azure \
    --service-cidr 10.0.0.0/16 \
    --dns-service-ip 10.0.0.10 \
    --docker-bridge-address 172.17.0.1/16 \
    --vnet-subnet-id $SUBNET_ID \
    --service-principal $SP_ID \
    --client-secret $SP_PASSWORD \
    --enable-addons monitoring
```

# Set the context to the new cluster

```
az aks get-credentials --resource-group $RESOURCE_GROUP_NAME --name $CLUSTER_NAME

```


# Deploy Helm

Deploy Helm

```
cat <<-EOF >helm-rbac.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system
EOF

kubectl apply -f helm-rbac.yaml

helm init --service-account tiller --node-selectors "beta.kubernetes.io/os"="linux"
```

Wait for Tiller to start and become ready (1/1)

```
watch kubectl get pods -n kube-system |grep tiller
```


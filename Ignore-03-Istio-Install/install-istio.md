# Download Istio

Specify the Istio version that will be leveraged throughout these instructions \
ISTIO_VERSION=1.1.3

```MacOS```
curl -sL "https://github.com/istio/istio/releases/download/$ISTIO_VERSION/istio-$ISTIO_VERSION-osx.tar.gz" | tar xz

```Linux```
curl -sL "https://github.com/istio/istio/releases/download/$ISTIO_VERSION/istio-$ISTIO_VERSION-linux.tar.gz" | tar xz

# Install Istioctl 

cd istio-$ISTIO_VERSION \
sudo cp ./bin/istioctl /usr/local/bin/istioctl \
sudo chmod +x /usr/local/bin/istioctl

# Install the Istio CRDs on AKS
helm install install/kubernetes/helm/istio-init --name istio-init --namespace istio-system

Verify the CRDs installed correctly

kubectl get jobs -n istio-system

kubectl get svc --namespace istio-system --output wide

# Configure GRAFANA andd KIALI to work with the Istio install


GRAFANA_USERNAME=$(echo -n "grafana" | base64) \
GRAFANA_PASSPHRASE=$(echo -n "REPLACE_WITH_YOUR_SECURE_PASSWORD" | base64) \

```
cat <<EOF | kubectl apply -f - 
apiVersion: v1 
kind: Secret 
metadata: 
  name: grafana 
  namespace: istio-system 
  labels: 
    app: grafana 
type: Opaque 
data: 
  username: $GRAFANA_USERNAME 
  passphrase: $GRAFANA_PASSPHRASE 
EOF
```


KIALI_USERNAME=$(echo -n "kiali" | base64) \
KIALI_PASSPHRASE=$(echo -n "REPLACE_WITH_YOUR_SECURE_PASSWORD" | base64) \

```
cat <<EOF | kubectl apply -f - \
apiVersion: v1 \
kind: Secret \
metadata: \
  name: kiali \
  namespace: istio-system \
  labels: \
    app: kiali \
type: Opaque \
data: \
  username: $KIALI_USERNAME \
  passphrase: $KIALI_PASSPHRASE \
EOF 
```

# Install Istio on AKS

helm install install/kubernetes/helm/istio --name istio --namespace istio-system \
  --set global.controlPlaneSecurityEnabled=true \
  --set mixer.adapters.useAdapterCRDs=false \
  --set grafana.enabled=true --set grafana.security.enabled=true \
  --set tracing.enabled=true \
  --set kiali.enabled=true


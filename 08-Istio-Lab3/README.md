# Demo Kiali - A service mesh observability dashboard is provided by Kiali.

kubectl port-forward -n istio-system $(kubectl get pod -n istio-system -l app=kiali -o jsonpath='{.items[0].metadata.name}') 20001:20001


## To use network policies, set up Calico:
```
minikube start --network-plugin=cni --cni=calico
#or 
kubectl apply -f https://docs.projectcalico.org/v3.23/manifests/calico.yaml
```
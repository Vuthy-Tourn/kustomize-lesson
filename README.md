## NOTE 

```bash 
kubectl kustomize ... 
kubectl version --client 

# external
kustomize version



# to render the values 
kubectl kustomize overlays/prod
kubectl kustomize overlays/dev 
kubectl apply -k  overlays/dev
kubectl kustomize overlays/dev > rendered.yaml 
kubectl apply | replace -f rendered.yaml 

```


## NOTE for example-002
```bash 
helm create sample-chart
```
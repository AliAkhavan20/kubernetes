# how to create an ingress-nginx to act as Reverse Proxy and dns Resolver

### First try to download the deploy.yaml file
```
wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.2/deploy/static/provider/baremetal/deploy.yaml
```
### change service type from clusterip TO LoadBalancer 
```
nano deploy.yaml
```
> Beware that if you dont have LoadBalancer cloud Provider then you must assign IP with Metallb tool , how to install Metallb has explained in other file
### Apply the changes
```
kubectl apply -f deploy.yaml
```

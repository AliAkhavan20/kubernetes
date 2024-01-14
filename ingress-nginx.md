# how to create an ingress-nginx to act as Reverse Proxy and dns Resolver

### First try to download the deploy.yaml file
```
wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.2/deploy/static/provider/baremetal/deploy.yaml
```
### change service type from ClusterIP TO LoadBalancer 
```
nano deploy.yaml
```
> Beware that if you dont have LoadBalancer cloud Provider then you must assign IP with Metallb tool , how to install Metallb has explained in other file
### Apply changes
```
kubectl apply -f deploy.yaml
```
### Create a nano file called ingress-nginx.yaml

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: minimal-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx-example
  rules:
  - http:
      paths:
      - path: /testpath
        pathType: Prefix
        backend:
          service:
            name: test
            port:
              number: 80
```
### Change port and domain name based on your needs.

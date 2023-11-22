# to assign external IP to frontend service run this command

```
kubectl patch svc frontend -p '{"spec":{"externalIPs":["192.168.74.194"]}}'
```
# proxy can be use as well
```
kubectl proxy --port=8001 --address='192.168.0.107' --accept-hosts="^*$"
```
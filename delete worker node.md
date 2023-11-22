# To remove a node follow the below steps

## Run on Master nodes
```
kubectl cordon <node-name>
```
```
kubectl drain <node-name> --force --ignore-daemonsets  --delete-emptydir-data
```
```
kubectl delete node <node-name>
```

## Run on Worker
```
kubeadm reset
```
```
rm -rf /etc/cni/net.dtc/cni/net.d
```
> ### The reset process does not clean your kubeconfig files and you must remove them manually.
> ###Please, check the contents of the $HOME/.kube/config file.
```
rm -rf /.kube/config
```
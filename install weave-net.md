# Weave-net is another CNI to use in kubernetes.
## we can use this CNI to set CIRD for out pods in kubernetes.
```
kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
```
> Once Weave is running in your cluster, you can interact with it by installing the Weave script on your nodes with these commands.
```
sudo curl -L git.io/weave -o /usr/local/bin/weave
```
```
sudo chmod a+x /usr/local/bin/weave
```
### Then call the script with commands such as weave status.

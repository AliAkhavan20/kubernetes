# controller-manager Connection refused:

##    Change bind-address (default: 127.0.0.1):
```
$ sudo vi /etc/kubernetes/manifests/kube-controller-manager.yaml
```
```
apiVersion: v1
kind: Pod
metadata:
  ...
spec:
  containers:
  - command:
    - kube-controller-manager
    ...
    - --bind-address=<your control-plane IP or 0.0.0.0>
    ...
```
> Do it for all of your masters
> If you are using control-plane IP, you need to change livenessProbe and startupProbe host, too.
> no need to reset or reboot just refresh prometheus and all is well.



# Kube-proxy Connection refused:

## Set the kube-proxy argument for metric-bind-address
```
$ kubectl edit cm/kube-proxy -n kube-system
```
```
...
kind: KubeProxyConfiguration
metricsBindAddress: 0.0.0.0:10249
...
```
```
$ kubectl delete pod -l k8s-app=kube-proxy -n kube-system
```
> every kube-proxy pod will be delete no further jobs to do.


# kube-scheduler Connection refused:

##    Change bind-address (default: 127.0.0.1):
```
$ sudo vi /etc/kubernetes/manifests/kube-scheduler.yaml
```
```
apiVersion: v1
kind: Pod
metadata:
  ...
spec:
  containers:
  - command:
    - kube-controller-manager
    ...
    - --bind-address=<your control-plane IP or 0.0.0.0>
    ...
```
> Do it for all of your masters
> If you are using control-plane IP, you need to change livenessProbe and startupProbe host, too.
> no need to reset or reboot just refresh prometheus and all is well.


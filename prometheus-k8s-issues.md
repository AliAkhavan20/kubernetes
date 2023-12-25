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
> If you are using control-plane IP, you need to change livenessProbe and startupProbe host, too.
> no need to reset or reboot just refresh prometheus and all is well.

# Use nerdctl to manage dockerd on your kubernetes master node

### Download Desired nerdctl from github
```
wget https://github.com/containerd/nerdctl/releases/download/v1.7.0/nerdctl-1.7.0-linux-amd64.tar.gz
```
```
wget https://github.com/containerd/nerdctl/releases/download/v1.7.0/nerdctl-full-1.7.0-linux-amd64.tar.gz
```
### Minimal Installation

#### Extract the archive to a path like /usr/local/bin or ~/bin .
```
tar Cxzvvf /usr/local/bin nerdctl-1.7.0-linux-amd64.tar.gz
```
### Full Installation

#### Extract the archive to a path like /usr/local or ~/.local .
```
tar Cxzvvf /usr/local nerdctl-full-1.7.0-linux-amd64.tar.gz
```
#### alias the ./nerdctl to be more easy to use
```
nano .bashrc
```
```
alias nerdctl='/usr/local/nerdctl'
```
#### apply bashrc
```
source ~/.bashrc
```
## Basic usage
To run a container with the default bridge CNI network (10.4.0.0/24):
```
nerdctl run -it --rm alpine
```
## Debugging Kubernetes
### To list local Kubernetes containers:
```
nerdctl --namespace k8s.io ps -a
```
### To build an image for local Kubernetes without using registry:
```
nerdctl --namespace k8s.io build -t foo /some-dockerfile-directory
```
```
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: foo
spec:
  containers:
    - name: foo
      image: foo
      imagePullPolicy: Never
EOF
```
### To load an image archive (docker save format or OCI format) into local Kubernetes:
```
nerdctl --namespace k8s.io load < /path/to/image.tar
```
### To read logs (experimental):
```
nerdctl --namespace=k8s.io ps -a
```
CONTAINER ID    IMAGE                                                      COMMAND                   CREATED          STATUS    PORTS    NAMES
...
e8793b8cca8b    registry.k8s.io/coredns/coredns:v1.9.3                     "/coredns -conf /etc…"    2 minutes ago    Up                 k8s://kube-system/coredns-787d4945fb-mfx6b/coredns
...
```
nerdctl --namespace=k8s.io logs -f e8793b8cca8b
```
[INFO] plugin/reload: Running configuration SHA512 = 591cf328cccc12bc490481273e738df59329c62c0b729d94e8b61db9961c2fa5f046dd37f1cf888b953814040d180f52594972691cd6ff41be96639138a43908
CoreDNS-1.9.3
linux/amd64, go1.18.2, 45b0a11
...

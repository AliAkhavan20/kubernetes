# Install multi-master Cluster

## Pre-requisites on all kubernetes nodes (masters & workers)

### Disable swap
```
swapoff -a; sed -i '/swap/d' /etc/fstab
```
### Firewall ports needed for communication

The following TCP ports are used by Kubernetes control plane components:
|PORTS|COMPONENTS|
|----|----|
|6443|Kubernetes API server|
|2379-2380|etcd server client API|
|10250|Kubelet API|
|10259|kube-scheduler|
|10257|kube-controller manager|

The following TCP ports are used by Kubernetes nodes:
|PORTS|COMPONENTS|
|----|----|
|10250|Kubelet API|
|30000-32767|NodePort Services|
```
sudo ufw allow 6443/TCP
```
#### Repeat the same command for other ports

### Enable and Load Kernel modules
```
cat >> /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF

modprobe overlay
modprobe br_netfilter
```
#### Add Kernel settings
```
cat >>/etc/sysctl.d/kubernetes.conf<<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
EOF

sysctl --system
```
#### Install containerd runtime
```
  apt update
  apt install -y containerd apt-transport-https
  mkdir /etc/containerd
  containerd config default > /etc/containerd/config.toml
  systemctl restart containerd
  systemctl enable containerd
```
#### Add apt repo for kubernetes
```
  curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
  apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"
```
#### Install Kubernetes components
```
  apt update
  apt install -y kubeadm=1.22.0-00 kubelet=1.22.0-00 kubectl=1.22.0-00
```
### Initialize Kubernetes Cluster on master1
```
kubeadm init --control-plane-endpoint="192.168.16.1:6443" --upload-certs --apiserver-advertise-address=192.168.16.2 --pod-network-cidr=10.10.0.0/16
```
#### Noted that Control-plane-endpoint wil be the Virtual ip that has been made in Keepalived service

### Install Calico network
```
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.4/manifests/tigera-operator.yaml
```
#### Remember to visit github calico all the time for instruction.
#### you can install the tiegra first and dont Deploy calico automatically.

### Custom Calico based on your CIDR
```
wget https://raw.githubusercontent.com/projectcalico/calico/v3.26.4/manifests/custom-resources.yaml
```
```
nano custom-resources.yaml
```
### change cidr to the subnet you configured
```
cidr: 10.10.0.0/16
```
### Apply the custom-resources file
```
kubectl create -f custom-resources
```

## Join other master nodes to the cluster
> Use the respective kubeadm join commands you copied from the output of kubeadm init command on the first master.

> IMPORTANT: Don't forget the --apiserver-advertise-address option to the join command when you join the other master nodes.

## master Pod Failures

> IMPORTANT: If other master pods keep failing add this to your host GRUB
```
nano /etc/default/grub
```
```
GRUB_CMDLINE_LINUX_DEFAULT="systemd.unified_cgroup_hierarchy=0"
```
```
update-grub
```
```
reboot
```
Enjoy Your Multi Master Cluster,Dont forget to run haproxy and keepalived i put in the repo so you wont have any failure accross your cluster

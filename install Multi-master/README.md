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
#### there will be a masater join command at the end of cluster init Copy the commands to join other master nodes.
#### Dont forget to use --api-advertise-address on each host for cluster joining

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







----------------------------------------------------
#Now its time to install kubernetes cluster one by one
after this 
go config control-plane1
go config calico cni
after that join other masters to control-plane1
one problem will cause and thats the masters will keep failing you should do this 
Try adding:

GRUB_CMDLINE_LINUX_DEFAULT="systemd.unified_cgroup_hierarchy=0"
To your /etc/default/grub file and then run update-grub, reboot and see if that helps.
----------------------------------------
----------------------------------------
----------------------------------------










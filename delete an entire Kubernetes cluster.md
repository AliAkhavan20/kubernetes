# Wipe-out a kubernetes cluster
To completely delete an entire Kubernetes cluster, you'll need to perform several steps depending on how the cluster was set up. The process may vary slightly based on the platform you used to create the cluster (e.g., a managed service like GKE, EKS, AKS, or a self-hosted cluster with kubeadm, kops, or another tool).
Below are generic steps you can follow. Please adjust them based on the specifics of your environment:

## Step 1: Delete Deployments, Services, and Pods
Use kubectl to delete all deployments, services, and pods in each namespace. Replace namespace-name with the actual namespace names used in your cluster.

```
kubectl delete deployment --all --namespace=namespace-name
```
```
kubectl delete service --all --namespace=namespace-name
```
```
kubectl delete pod --all --namespace=namespace-name
```
### Repeat this for each namespace in your cluster.

## Step 2: Delete Persistent Volumes and Persistent Volume Claims
```
kubectl delete persistentvolumes --all
kubectl delete persistentvolumeclaims --all
```
## Step 3: Delete ConfigMaps and Secrets
```
kubectl delete configmaps --all --namespace=namespace-name
kubectl delete secrets --all --namespace=namespace-name
```
## Step 4: Delete Custom Resources (if any)
If you have installed custom resources (e.g., CustomResourceDefinitions), delete them:
```
kubectl get customresourcedefinitions.apiextensions.k8s.io
```
```
kubectl delete customresourcedefinition <custom-resource-name>
```
## Step 5: Delete Namespaces
```
kubectl delete namespace --all
```
## Step 6: Reset kubeadm (for self-hosted clusters)

###If you used kubeadm to set up your cluster, you might need to reset each node:

## On each worker node
```
sudo kubeadm reset
```
## On the master node
```
sudo kubeadm reset --force
```
## Step 7: Remove kubectl Configuration
### Remove the kubectl configuration file (usually located at ~/.kube/config):
```
rm ~/.kube/config
```
## Step 8: Remove Docker or Containerd (optional)
### If you installed Docker or Containerd manually, you might want to remove them:

## For Docker
```
sudo apt-get purge docker-ce
```
## For Containerd
```
sudo apt-get purge containerd
```
## Step 9: Remove Kubernetes Binaries (optional)
###Remove Kubernetes binaries (for self-hosted clusters):
```
sudo rm -rf /etc/kubernetes
sudo rm -rf /var/lib/kubelet
```
## Step 10: Remove etcd Data (for self-hosted clusters)
###If etcd is used as the backend storage, remove the etcd data:
```
sudo rm -rf /var/lib/etcd
```
## Step 11: Remove Network Plugins
### If you used a specific network plugin (e.g., Calico, Flannel), follow the plugin's documentation to remove it.

> Note:
> Ensure that you have backups or snapshots before performing any deletion, especially in production environments. Deleting resources is irreversible.

> The steps provided are general, and you might need to adjust them based on the specifics of your setup. If you used a cloud-managed Kubernetes service (e.g., GKE, EKS, AKS), refer to the respective cloud provider's documentation for additional steps.

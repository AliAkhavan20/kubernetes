﻿# Setup prometheus monitoring for kubernetes Guide

## Create a Namespace & ClusterRole
First, we will create a Kubernetes namespace for all our monitoring components. If you don’t create a dedicated namespace, all the Prometheus kubernetes deployment objects get deployed on the default namespace.

### Execute the following command to create a new namespace named monitoring.
```
kubectl create namespace monitoring
```
Prometheus uses Kubernetes APIs to read all the available metrics from Nodes, Pods, Deployments, etc. For this reason, we need to create an RBAC policy with read access to required API groups and bind the policy to the monitoring namespace.

## Step 1: Create a file named clusterRole.yaml and copy the following RBAC role.

Note: In the role, given below, you can see that we have added get, list, and watch permissions to nodes, services endpoints, pods, and ingresses. The role binding is bound to the monitoring namespace. If you have any use case to retrieve metrics from any other object, you need to add that in this cluster role.
```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: prometheus
rules:
- apiGroups: [""]
  resources:
  - nodes
  - nodes/proxy
  - services
  - endpoints
  - pods
  verbs: ["get", "list", "watch"]
- apiGroups:
  - extensions
  resources:
  - ingresses
  verbs: ["get", "list", "watch"]
- nonResourceURLs: ["/metrics"]
  verbs: ["get"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: prometheus
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: prometheus
subjects:
- kind: ServiceAccount
  name: default
  namespace: monitoring
```
## Step 2: Create the role using the following command.
```
kubectl create -f clusterRole.yaml
```
Create a Config Map To Externalize Prometheus Configurations
All configurations for Prometheus are part of prometheus.yaml file and all the alert rules for Alertmanager are configured in prometheus.rules.
prometheus.yaml: This is the main Prometheus configuration which holds all the scrape configs, service discovery details, storage locations, data retention configs, etc)
prometheus.rules: This file contains all the Prometheus alerting rules
By externalizing Prometheus configs to a Kubernetes config map, you don’t have to build the Prometheus image whenever you need to add or remove a configuration. You need to update the config map and restart the Prometheus pods to apply the new configuration.
> The config map with all the Prometheus scrape config and alerting rules gets mounted to the Prometheus container in /etc/prometheus location as prometheus.yaml and prometheus.rules files.

## Step 1: Create a file called config-map.yaml and copy the file contents from this link –> Prometheus Config File.

## Step 2: Execute the following command to create the config map in Kubernetes.
```
kubectl create -f config-map.yaml
```
It creates two files inside the container.

> Note: In Prometheus terms, the config for collecting metrics from a collection of endpoints is called a job.

> The prometheus.yaml contains all the configurations to discover pods and services running in the Kubernetes cluster dynamically. We have the following scrape jobs in our Prometheus scrape configuration.

> kubernetes-apiservers: It gets all the metrics from the API servers.
> kubernetes-nodes: It collects all the kubernetes node metrics.
> kubernetes-pods: All the pod metrics get discovered if the pod metadata is annotated with prometheus.io/scrape and prometheus.io/port annotations.
> kubernetes-cadvisor: Collects all cAdvisor metrics.
> kubernetes-service-endpoints: All the Service endpoints are scrapped if the service metadata is annotated with prometheus.io/scrape and prometheus.io/port annotations. It can be used for black-box monitoring.
> prometheus.rules contains all the alert rules for sending alerts to the Alertmanager.

## Create a Prometheus Deployment
### Step 1: Create a file named prometheus-deployment.yaml and copy the following contents onto the file. 
In this configuration, we are mounting the Prometheus config map as a file inside /etc/prometheus as explained in the previous section.

> Note: This deployment uses the latest official Prometheus image from the docker hub. Also, we are not using any persistent storage volumes for Prometheus storage as it is a basic setup. When setting up Prometheus for production uses cases, make sure you add persistent storage to the deployment.
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prometheus-deployment
  namespace: monitoring
  labels:
    app: prometheus-server
spec:
  replicas: 1
  selector:
    matchLabels:
      app: prometheus-server
  template:
    metadata:
      labels:
        app: prometheus-server
    spec:
      containers:
        - name: prometheus
          image: prom/prometheus
          args:
            - "--storage.tsdb.retention.time=12h"
            - "--config.file=/etc/prometheus/prometheus.yml"
            - "--storage.tsdb.path=/prometheus/"
          ports:
            - containerPort: 9090
          resources:
            requests:
              cpu: 500m
              memory: 500M
            limits:
              cpu: 1
              memory: 1Gi
          volumeMounts:
            - name: prometheus-config-volume
              mountPath: /etc/prometheus/
            - name: prometheus-storage-volume
              mountPath: /prometheus/
      volumes:
        - name: prometheus-config-volume
          configMap:
            defaultMode: 420
            name: prometheus-server-conf
  
        - name: prometheus-storage-volume
          emptyDir: {}
```
### Step 2: Create a deployment on monitoring namespace using the above file.
```
kubectl create  -f prometheus-deployment.yaml 
```
### Step 3: You can check the created deployment using the following command.
```
kubectl get deployments --namespace=monitoring
```
## Connecting To Prometheus Dashboard
You can view the deployed Prometheus dashboard in three different ways.

> Using Kubectl port forwarding
> Exposing the Prometheus deployment as a service with NodePort or a Load Balancer.
> Adding an Ingress object if you have an Ingress controller deployed.

### Method 1: Using Kubectl port forwarding
Using kubectl port forwarding, you can access a pod from your local workstation using a selected port on your localhost. This method is primarily used for debugging purposes.

### Step 1: First, get the Prometheus pod name.
```
kubectl get pods --namespace=monitoring
```
#### The output will look like the following.
```
➜  kubectl get pods --namespace=monitoring
...
NAME                                     READY     STATUS    RESTARTS   AGE
prometheus-monitoring-3331088907-hm5n1   1/1       Running   0          5m
...
```
### Step 2: Execute the following command with your pod name to access Prometheus from localhost port 8080.

> Note: Replace prometheus-monitoring-3331088907-hm5n1 with your pod name.
```
kubectl port-forward prometheus-monitoring-3331088907-hm5n1 8080:9090 -n monitoring
```
### Step 3: Now, if you access http://localhost:8080 on your browser, you will get the Prometheus home page.

### Method 2: Exposing Prometheus as a Service [NodePort & LoadBalancer]
To access the Prometheus dashboard over a IP or a DNS name, you need to expose it as a Kubernetes service.

### Step 1: Create a file named prometheus-service.yaml and copy the following contents. We will expose Prometheus on all kubernetes node IP’s on port 30000.

Note: If you are on AWS, Azure, or Google Cloud, You can use Loadbalancer type, which will create a load balancer and automatically points it to the Kubernetes service endpoint.
```
apiVersion: v1
kind: Service
metadata:
  name: prometheus-service
  namespace: monitoring
  annotations:
      prometheus.io/scrape: 'true'
      prometheus.io/port:   '9090'
spec:
  selector: 
    app: prometheus-server
  type: NodePort  
  ports:
    - port: 8080
      targetPort: 9090 
      nodePort: 30000
```
The annotations in the above service YAML makes sure that the service endpoint is scrapped by Prometheus. The prometheus.io/port should always be the target port mentioned in service YAML

### Step 2: Create the service using the following command.
```
kubectl create -f prometheus-service.yaml --namespace=monitoring
```
### Step 3: Once created, you can access the Prometheus dashboard using any of the Kubernetes node’s IP on port 30000. If you are on the cloud, make sure you have the right firewall rules to access port 30000 from your workstation.

### Step 4: Now if you browse to status --> Targets, you will see all the Kubernetes endpoints connected to Prometheus automatically using service discovery as shown below.

The kube-state-metrics down is expected and I’ll discuss it shortly.

### Step 5: You can head over to the homepage and select the metrics you need from the drop-down and get the graph for the time range you mention. An example graph for container_cpu_usage_seconds_total is shown below.

### Method 3: Exposing Prometheus Using Ingress
If you have an existing ingress controller setup, you can create an ingress object to route the Prometheus DNS to the Prometheus backend service.

Also, you can add SSL for Prometheus in the ingress layer. You can refer to the Kubernetes ingress TLS/SSL Certificate guide for more details.

Here is a sample ingress object. Please refer to this GitHub link for a sample ingress object with SSL
```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: prometheus-ui
  namespace: monitoring
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
  # Use the host you used in your kubernetes Ingress Configurations
  - host: prometheus.example.com
    http:
      paths:
      - backend:
          serviceName: prometheus-service
          servicePort: 8080
```
## Setting Up Kube State Metrics
Kube state metrics service will provide many metrics which is not available by default. Please make sure you deploy Kube state metrics to monitor all your kubernetes API objects like deployments, pods, jobs, cronjobs etc.

### Step 1: Clone the Github repo
```
git clone https://github.com/devopscube/kube-state-metrics-configs.git
```
### Step 2: Create all the objects by pointing to the cloned directory.
```
kubectl apply -f kube-state-metrics-configs/
```
### Step 3: Check the deployment status using the following command.
```
kubectl get deployments kube-state-metrics -n kube-system
```

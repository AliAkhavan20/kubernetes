Step-1. We need to add the Helm Stable Charts for your local client. Execute the below command:

helm repo add stable https://charts.helm.sh/stable

Step2: Add Prometheus Helm repo

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

Step 3: Create Prometheus namespace

kubectl create namespace prometheus

helm install stable prometheus-community/kube-prometheus-stack -n prometheus

    above command is used to install kube-Prometheus-stack. The helm repo kube-stack-Prometheus comes with a Grafana deployment embedded ( as the default one ).

Now Prometheus is installed using helm in the ec2 instance

    To check whether Prometheus is installed or not use the below command

kubectl get pods -n prometheus

to check the services file (svc) of the Prometheus

kubectl get svc -n prometheus

Grafana will be coming along with Prometheus as the stable version
This output is conformation that our Prometheus is installed successfully
there is no need of installing Grafana as a separate tool it comes along with Prometheus

let’s expose Prometheus and Grafan to the external world
there are 2 ways to expose

    through Node Port
    through LoadBalancer

let’s go with the LoadBalancer
to attach the load balancer we need to change from ClusterIP to LoadBalancer
command to get the svc file

kubectl edit svc stable-kube-prometheus-sta-prometheus -n prometheus

change it from Cluster IP to LoadBalancer after changing make sure you save the file

you guys can see I have a load balancer for my Prometheus which I can access from that link

we can use a Prometheus UI for monitoring the EKSbut the UI of Prometheus is not a convent for the user Grafana will extract the matrix from the Prometheus UI and show it in a user-friendly manner

let’s change the SVC file of the Grafana and expose it to the outer world

command to edit the SVC file of grafana

kubectl edit svc stable-grafana -n prometheus

the Grafana LoadBalancer also exposed

use the link of the LoadBalancer and access from the Browser

kubectl get secret --namespace prometheus stable-grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo

use the above command to get the password
the user name is admin
Create a Dashboard in Grafana

let’s create a Dashboard by importing
click on Import and import the dashboard with numbers

there are plenty of ready templates to use the pre-existing templates and modify based on our desired
it uses a Prometheus

click on the Import


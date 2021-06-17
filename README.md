##Setting up the environment##

*To start using the service, first we need to set up the following*

AWS CLI — The AWS Command Line Interface (CLI) is a unified tool to manage your AWS services. With just one tool to download and configure, you can control multiple AWS services from the command line and automate them through scripts. Click here for the complete information about setting up AWS CLI.
eskctl — It is a simple CLI tool for creating clusters on EKS — Amazon’s new managed Kubernetes service for EC2. It is written in Go, uses CloudFormation, was created by Weaveworks and it welcomes contributions from the community.
kubectl — kubectl controls the Kubernetes cluster manager. Click here for the complete information about setting up kubectl.
#cluster creation#
use this command: 
eksctl create cluster -f cluster.yml
#wordpress-mysql-deployment#
navigate to the cloned folder and execute:
kubectl create -k .
#Deploy Prometheus#
We will use gp2 EBS volumes for simplicity and demonstration purpose. When deploying in production, you would use io1 volumes with desired IOPS and increase the default storage size in the manifests to get better performance. Run the following command:
**
kubectl create namespace prometheus

helm install prometheus prometheus-community/prometheus \
    --namespace prometheus \
    --set alertmanager.persistentVolume.storageClass="gp2" \
    --set server.persistentVolume.storageClass="gp2"
*Check if Prometheus components deployed as expected*
kubectl get all -n prometheus
**
kubectl port-forward -n prometheus deploy/prometheus-server 8080:9090
**
*Deployment of Grafana*
kubectl create namespace grafana

helm install grafana grafana/grafana \
    --namespace grafana \
    --set persistence.storageClassName="gp2" \
    --set persistence.enabled=true \
    --set adminPassword='MkolweAwesome15@' \
    --values ${HOME}/wordpress-mysql-prometheus-grafana/grafana.yaml \
    --set service.type=LoadBalancer
*checking resources deployment in grafana namespace*
kubectl get all -n grafana
*When logging in, use the username admin and get the password hash by running the following:*
kubectl get secret --namespace grafana grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
#Cluster Monitoring Dashboard

For creating a dashboard to monitor the cluster:

Click '+' button on left panel and select ‘Import’.
Enter 3119 dashboard id under Grafana.com Dashboard.
Click ‘Load’.
Select ‘Prometheus’ as the endpoint under prometheus data sources drop down.
Click ‘Import’.
#Pods Monitoring Dashboard

For creating a dashboard to monitor all the pods:

Click '+' button on left panel and select ‘Import’.
Enter 6417 dashboard id under Grafana.com Dashboard.
Click ‘Load’.
Enter Kubernetes Pods Monitoring as the Dashboard name.
Click change to set the Unique identifier (uid).
Select ‘Prometheus’ as the endpoint under prometheus data sources drop down.s
Click ‘Import’.










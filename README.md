Hello Host IP Application
Project Definition :

Creating a greenfield OPS environment and deploy an application to displays IP address of the host machine.

System Architecture : 

Application is running on a 2-tiers architecture - running Web-server and Application is different tiers with Load-balancer service for High Availability. In addition this provides additional security to Application server from being exposed to external access — running them behind private cluster IP Service.

Kubernetes Cluster is deployed with a NGINX server running Load Balancer service and 3 replicas of Nodejs Application Server with Cluster IP Service.

Application Server and Web-server images are created and deployed as Docker Images

Helm is used to create Application and Web server charts with parameters to automate orchestration of Kubernetes clusters running on Minikube.

Terraform code will automate the deployment of Kubernetes Cluster with IaC pipeline.

Minikube Dashboard to view the deployment

Helm Charts
Features :

High Availability - Application runs on 3 replicas increasing availability. Also increases Performance of application when heavy load.

Scale on demand - Rescaling is faster (change replica count in values.yaml) and requires no down-time

Self-healing - Kubernetes replaces pods in case of failure, upgrades with rolling updates and application can self heal in case node failure - supporting “zero-downtime” rolling updates

Portability - Run anywhere that has Kubernetes — whether in cloud AWS Cloud using EMR service, Google, Azure cloud, or on-premise datacenter. Currently application is running on on single system.

Automation - Using Helm charts and Terraform Devops tools automates the deployment across multiple environments and platforms.
Project Components : 

Nodejs Application
NGINX Server
Docker Images
Kubernetes Cluster (MiniKube)
Helm Chart
Terraform

Approach : Developed first Application and Server deployments and run them successful, then converted to Helm Chart. Then created IaC with terraform.
Implementation Steps : 

1. Nodejs application which return the IP address of host on which application is running
2. Application Server which runs Nodejs app script
3. App-server Docker image create for Nodejs application files to install Nodejs and run application server
4. Kubernetes cluster with Deployment, Service and pods created from the docker image
5. Application service run in minikube container 

Webserver 

6. WebServer - NGINX server which runs as a Loadbalancer with External IP
7. Web-server Docker image create for NGINX server files
8. Kubernetes cluster with Deployment, Service and pods created from the docker image
9. Application service run in minikube container 

10. Helm Chart is used to orchestrate deployment of Kubernetes Cluster

11. Terraform to automate creation of Kubernetes Cluster. (Node : Terraform does not create Helm charts yet)

Technical Stack used in the Project :

For Application with Cluster IP services : Nodejs with express, networkInterfaces modules
For Web-server with Loadbalancer services : NGINX server

Docker for containerization of Application and Server deployments

Minikube cluster running in docker virtual machine - to deploy containerized application to Kubernetes cluster

Kubernetes(K8s) for automating deployment, scaling and management of Application and Server deployment containers

Helm for creating parameterized charts of Web-server and App-server applications and orchestration of Kubernetes cluster

Minikube dashboard - user interface to view Kubernetes deployments by traffic light
Terraform - to automate kubernetes deployment with Infrastructure as code tool


Folder Structure :

jvp-hostip-app (root folder)
--app (application folder)
----app.js (nodejs application code)
----Dockerfile
----package.json (nodejs package)
--server (server folder)
----default.conf (nginx server configuration file)
----Dockerfile

--helm
----templates
------app-deployment.yaml
------app-service.yaml
———values.yaml



Security 
a.What Security considerations did you think about (secret storage, https, network security, etc) 
I am using Secrets to pass security configuration (credentials and private keys) to application. 
Secrets are similar to Config Map - and can be injected either using “environment variables” or “by mounting files”

Here I am using environment variables.

Secrets are distributed in pods running application and not written to physical storage keeping them secure, stored on master node as encrypted.  We retrieve them in “Base64” encoded form. 

Base64 encoding example : 
Secrets yaml file :












Adding secrets as “Environment Variables” in app deployment file














Ssecure application from common web attacks like DDOS, Cross-site scripting, SQL Injection by placing application servers with private “ClusterIP” behind LoadBalancer 

With current design - we don’t want application server to be exposed directly to external traffic -- all request come through Loadbalancer. 
Loadbalancer server can also be used for authentication and to funnel traffic through a single path. Also experimented with all three types of services - ClusterIP, NodePort and LoadBalancer.

I am also using container environment variables in values.yaml file.
I created Application Readiness and Liveness Probes to monitor health check of cluster services. Used in second version of application. Configured health-check within nodes application.

If deploying cluster on AWS - server credentials can be stored in “Secret Manager” or “Parameter Store”.


 
Used Resource manifest --- controls deployment - what container to run and how many instance


How to build and use the project
Please refer to appendix for detailed step on Installations required for developing and running the project. 

Without Terraform :

Steps :
Once code is created for app, create docker image in minikube docker container : and push it to docker hub

Root folder : jvp-hostip-app
Navigate to jvp-hostip-app/app folder and run command to create docker image

Docker login
run: docker login
run: eval $(minikube docker-env)
Cd app
run: docker build . -f Dockerfile -t jvp-appserver:latest


2. Once config is created for server, create docker image in minikube docker container :
Navigate to root folder : jvp-hostip-app
run: docker build . -f Dockerfile -t jvp-webserver:latest


3. Confirm minikube is running 
Run: Minikube start


run: Minikube dashboard

4. Create kubectl namespace

 run: kubectl create namespace jvpnamespace


5. Run Helm chart to create cluster, deployment, pods and services

Confirm in root folder: jvp-hostip-app
run: helm install --namespace jvpnamespace jvp-hostip-app ./helm
Confirm helm : helm list
To upgrade : 
Run:helm upgrade --namespace jvpnamespace jvp-hostip-app ./helm

6. Confirm deployment

Confirm pods created :

Run:Kubectl get pods

Or 
Run : minikube dashboard 
It will open web browser interface as below :
Pods : 

Replica Set :


7. Run loadbalancer service :

minikube service jvp-webserver-loadbalancer-service -n jvpnamespace





Result : a browser window will open to display result






8. Output :

One time I was able to direct nginx to Apiserver — but then couldn’t redirect service to Apiserver next time due to network poxy port issue.


WITH Terraform :

Folder - jvp-terraform

Navigate to folder jvp-terraform

Check parameters in inputs.tfvars files (personalize as required)
Run : terraform init (to initialize terraform providers)
Run : terraform plan -var-file=inputs.tfvars
Run : terraform apply -var-file=inputs.tfvars


Health-check and Monitoring :

Configured Liveness and Readiness probes to monitor health of services and configured rules in deployment file. 




















Discussions : 

b.What you would do if you had more time
	
If there was more time — I would
	- add custom code for liveness and readiness probe
	- automate deployment of application on AWS/Google cloud containers
	- experiment on more steps on securing application
	- convert 2 tier application to 3 tier application for separating web-application-database layers
	- debugging more on networking aspect of Kubernetes

c.How would you improve the deployment process

Currently using Helm and Terraform ways of deployment. I will like to experiment with both Terraform and Helm together to discover additional advantages and further optimize the deployment.
Experiment with rolling update - try to create “one touch deployment” process with Gitlab DevOps pipelines for automated deployment and easy rollback. 
Adding auditing capabilities as its required once application is larger and more complex as a next step.


d.What's broken about ops, and how would you fix it?

Currently a networking aspect of how request and response flows between nginx server and application through cluster ip interface is not very clear. I will like to fix that part. 
Also try adding liveness and readiness probes to application for better monitoring.


Conclusion :
This is a very simple multi-tier application running at a small scale which creates prototype of High-availability application using Kubernetes, Helm and IaC (Terraform) - featuring the benefits of this design like High Availability, Portability, Performance, Scalability and Devops Automation.



Appendix

Installation Steps : 

Nodejs Application :

Used Express package to handle request and response to nodejs application

“use strict” — to indicate that code should be executed in strict mode and does not have any variables undefined

Installation step (without Terraform)

Installed Nodejs - Version v16.12.2

Docker - 4.4.2

Kubectl (mac-intel) 

kubectl controls the Kubernetes cluster manager

Installation :  https://kubernetes.io/docs/tasks/tools/install-kubectl-macos/


4, Install minikube

 
minikube is local Kubernetes, focusing on making it easy to learn and develop for Kubernetes.
Here it is run on Docker

5. Install Helm Chart 
Helm v3.8.0-rc.2
https://helm.sh/docs/intro/install/
 
$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 $ chmod 700 get_helm.sh $ ./get_helm.sh


Use Helm to create resource manifest which controls deployment - what container to run and how many instances.

Debugging : ssh into pods 

kubectl exec -it jvp-webserver-deployment-68589cdcd9-bq5f6  -- sh
kubectl --namespace jvpnamespace port-forward $POD_NAME 8080:$CONTAINER_PORT
kubectl get deployments -n jvpnamespace
kubectl get pods -n jvpnamespace

Bug with tool : https://github.com/kubernetes/minikube/issues/549

Figure 1 Application Architecture 

![image](https://github.com/user-attachments/assets/5499a612-68b8-4c7b-ab2e-cef3b92f7a18)

Continuous Integration: register-app
<br>
Continuous Delivery:cd-register-app
<br>
https://github.com/sirisha-k83/cd-register-app.git

Deploying a simple java project on EKS Cluster with CI/CD

 1. GitHub + Jenkins Integration
Webhooks notify Jenkins on GitHub commits.

Jenkins must be public; use http://<jenkins-url>/github-webhook/ in GitHub settings.

Enable "GitHub hook trigger for GITScm polling" in Jenkins Pipeline.

Required Jenkins plugins:

GitHub Plugin

Git Plugin

Pipeline Plugin

GitHub Integration (optional)

⚙️ 2. Jenkins Setup
Use "Pipeline script from SCM" for Git repo.

Add Docker, Maven, Java, and SonarQube to Jenkins.

Install Maven and JDK (e.g., Maven 3.6.3, OpenJDK 17).

Configure Jenkins for Docker access:
sudo usermod -aG docker jenkins

🔒 3. SonarQube Setup
Run with Docker:

docker run -d --name sonarqube -p 9000:9000 sonarqube
Create token in SonarQube and add it as a Jenkins secret.


📬 4. Email Notifications in Jenkins
Configure SMTP settings (e.g., Gmail).

Use app-specific password for secure access.

Send a test email to verify.

🐳 5. Dockerfile Guidelines
Base images:

Java WAR: tomcat:9-jdk11

Spring Boot: openjdk:17-jdk

Node.js: node:18

🚀 6. Continuous Delivery Pipeline
Jenkins builds Docker image and pushes it to Azure Container Registry (ACR).

Set up AKS cluster and connect to ACR:

az aks update --name myAKSCluster --resource-group <resourcegroup_name> --attach-acr <ACR_name>

Add Jenkins SP (Service Principal) with AcrPush role to access ACR.

☸️ 7. Argo CD for CD
Watch Git repo for Kubernetes manifests.

Auto-syncs changes (like image tag updates) into AKS.

Install Argo CD via:

kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

📊 8. Monitoring with Prometheus + Grafana
Install via Helm:

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install kube-monitoring prometheus-community/kube-prometheus-stack
Default Grafana credentials:

Username: admin

Password: prom-operator

Use dashboards like:

Node Exporter Full

Cluster Overview

API Server metrics






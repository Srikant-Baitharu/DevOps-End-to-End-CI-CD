# 🚀 End-to-End Production-Grade CI/CD Pipeline with DevSecOps on Kubernetes

This project implements a complete production-level CI/CD pipeline with DevSecOps practices, infrastructure provisioning using Terraform, and containerized application deployment to a Kubernetes cluster. The system ensures secure, automated, and scalable delivery using Jenkins, GitHub, Trivy, SonarQube, and monitoring tools like Prometheus and Grafana.

---

## 🧰 Tech Stack

- **Version Control**: Git, GitHub  
- **CI/CD Tool**: Jenkins  
- **Build Tool**: Maven  
- **Code Quality**: SonarQube  
- **Security Scan**: Trivy  
- **Infrastructure as Code**: Terraform  
- **Containerization**: Docker  
- **Orchestration**: Kubernetes  
- **Monitoring & Alerting**: Prometheus, Grafana   
- **Container Registry**: DockerHub  
- **Application Repo**: Spring Boot/Microservice App (Maven based)

---

## 📌 Workflow Overview

1. 👨‍💻 Developer pushes code to GitHub.
2. ☕ Jenkins triggers the pipeline:
   - Pulls code from GitHub.
   - Builds the project with Maven.
   - Performs code analysis using SonarQube.
   - Scans the Docker image with Trivy.
3. 🐳 Docker images are pushed to DockerHub.
4. ☁️ Terraform provisions infrastructure (EKS, etc.) on AWS (Manually).
5. 🚀 Jenkins deploys the application to Kubernetes.
6. 🔒 Sealed Secrets (if used) manage Kubernetes secrets securely.
7. 📈 Prometheus + Grafana setup for monitoring metrics and health.

---

# 1️⃣ Create and Configure Git Repository

# Create a new GitHub repo
# Push your application source code
git init
git remote add origin <repo-url>
git add .
git commit -m "Initial commit"
git push origin main

# 2️⃣ EC2 Instances Setup
Service	    Instance Type	   Storage	   Notes
SonarQube	  t2.medium	      default	   Minimum 4GB RAM
Nexus	        t2.medium	      default  	Minimum 4GB RAM
Jenkins	     t2.large	      25 GB    	Main CI/CD engine
Terraform	  t2.medium	      default	   For EKS provisioning

# 3️⃣ Security Groups & Networking
Create a Security Group with inbound rules for required ports:
22 (SSH)
8080 (Jenkins)
8081 (Nexus)
9000 (SonarQube)
3000 (Grafana)
9090 (Prometheus)
9115 (Blackbox Exporter)
80 / 443 (HTTP/HTTPS if needed)

# 4️⃣ Connect Servers with MobaXterm
Install MobaXterm
Enable SSH KeepAlive for stable sessions
Connect to all servers via public IP

# 5️⃣ In JENKINS Server
sudo apt update
sudo apt install openjdk-17-jre-headless -y

vi 1.sh
# Add Jenkins installation commands:
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins -y

sudo chmod +x 1.sh
./1.sh

# 6️⃣ Install docker in jenkins server
sudo apt install docker.io -y
sudo chmod 666 /var/run/docker.sock

http://<JENKINS_PUBLIC_IP>:8080
sudo cat /var/lib/jenkins/secrets/initialAdminPassword

# 7️⃣ Install Jenkins Plugins
SonarQube Scanner
Maven Integration
Config File Provider
Pipeline Maven Integration
Kubernetes Plugins (CLI, Credentials, API, Client)
Docker Pipeline
Eclipse Temurin Installer
Pipeline Stage View

# 8️⃣ Install Trivy on Jenkins Server
sudo apt-get install wget apt-transport-https gnupg lsb-release -y
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy -y

# 9️⃣ Nexus Installation
sudo apt update
sudo apt install docker.io -y
sudo docker run -d -p 8081:8081 sonatype/nexus3
sudo docker ps

http://<NEXUS_PUBLIC_IP>:8081

# Get admin password:
sudo docker exec -it <CONTAINER_ID> /bin/bash
cd sonatype-work/nexus3
cat admin.password

# 🔟 SonarQube Installation
sudo apt update
sudo apt install docker.io -y
sudo docker run -d -p 9000:9000 sonarqube:lts-community
sudo docker ps

http://<SONARQUBE_PUBLIC_IP>:9000
Username: admin
Password: admin

# 1️⃣1️⃣Now next part of our project that is deployment (aws eks)

# Create instance named terra-instance for terraform server t2.medium

sudo apt update
now install aws cli
curl "https://awscli.amazon.aws.com/awscli-exe-linux_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install

create access key for cli iam user

aws configure

Now install terraform
sudo snap install terraform --classic
after installation:
mkdir terra
cd terra
vi main.tf (put all code )
vi output.tf (put all code)
vi variables.tf(put all code)
put sudo before this command if any error occurs

terraform init
terraform plan
terraform apply --auto-approve

kubectl get nodes
sudo snap install kubectl --classic

Now cluster is already created with help of iac(terraform)
so now we have to connect it
for connecting use this command
aws eks --region ap-south-1 update-kubeconfig --name cluster-name
kubectl get nodes

come out of all folder
create a service acc
vi svc.yml
apiVersion: v1
kind: ServiceAccount
metadata: 
	name: Jenkins
	namespace: webapps

kubectl create ns webapps
kubectl apply -f svc.yml

create a role
sudo vi role.yml -->
-------------------------------------------------------------------------------
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: app-role
  namespace: webapps
rules:
  - apiGroups:
        - ""
        - apps
        - autoscaling
        - batch
        - extensions
        - policy
        - rbac.authorization.k8s.io
    resources:
      - pods
      - secrets
      - componentstatuses
      - configmaps
      - daemonsets
      - deployments
      - events
      - endpoints
      - horizontalpodautoscalers
      - ingress
      - jobs
      - limitranges
      - namespaces
      - nodes
      - pods
      - persistentvolumes
      - persistentvolumeclaims
      - resourcequotas
      - replicasets
      - replicationcontrollers
      - serviceaccounts
      - services
    verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
------------------------------------------------------------------------------------------
kubectl apply -f role.yml

//Now bind the role to service account
sudo vi bind.yml  -->
------------------------------------------------------------------------------------------
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: app-rolebinding
  namespace: webapps 
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: app-role 
subjects:
- namespace: webapps 
  kind: ServiceAccount
  name: jenkins 
-------------------------------------------------------------------------------------------
kubectl apply -f bind.yml

--generate token--
sudo vi jen-secret.yml  -->
-------------------------------------------------------------------------------------------
apiVersion: v1
kind: Secret
type: kubernetes.io/service-account-token
metadata:
  name: mysecretname
  annotations:
    kubernetes.io/service-account.name: myserviceaccount
-------------------------------------------------------------------------------------------

kubectl apply -f jen-secret.yml -n webapps

kubectl create secret docker-registry

kubectl describe secret mysecretname -n webapps


//Now add credentials in jenkins
Modify pipeline




--- PROMETHEUS , GRAFANA AND BLACKBOX EXPORTER INSTALLATION IN SAME SERVER ---

--PROMETHEUS--

wget https://github.com/prometheus/prometheus/releases/download/v3.4.2/prometheus-3.4.2.linux-amd64.tar.gz

ls

tar -xvf prometheus-3.4.2.linux-amd64.tar.gz  // extract file

rm .tar.gz file

mv prometheus-3.4.2.linux-amd64/ prometheus  // rename

cd prometheu/

./prometheus &                               // to run on port 9090 (default)

--BLACKBOX--

wget URL

tar -xvf .tar.gzfile

rm .tar.gzfile

mv blackbox...amd64/ blackbox

cd blackbox/

./blackbox_exporter &                       // to run on port 9115    & -> means it will run in background

--GRAFANA--

sudo apt-get install -y adduser libfontconfig1 musl

wget https://dl.grafana.com/enterprise/release/grafana-enterprise_12.0.2_amd64.deb

sudo dpkg -i grafana-enterprise_12.0.2_amd64.deb

sudo /bin/systemctl start grafana-server    // it will run on port 3000 



---for access data--
cd prometheus
ls
vi prometheus.yml
add this ->
- job_name: 'blackbox'
    metrics_path: /probe
    params:
      module: [http_2xx]  # Look for a HTTP 200 response.
    static_configs:
      - targets:
        - http://prometheus.io    # Target to probe with http.
        - https://prometheus.io   # Target to probe with https.
        - http://example.com:8080 # Target to probe with http on port 8080.
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: 127.0.0.1:9115

insert link which we want to monitor that is our site URL

all can access by <ec2 instance ip>:port
grafana=3000
blackbox=9115
prometheus=9090




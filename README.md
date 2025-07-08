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
4. ☁️ Terraform provisions infrastructure (EKS, etc.) on AWS.
5. 🚀 Jenkins deploys the application to Kubernetes.
6. 🔒 Sealed Secrets (if used) manage Kubernetes secrets securely.
7. 📈 Prometheus + Grafana setup for monitoring metrics and health.

---




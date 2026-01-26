
## Advanced End-to-End DevSecOps Kubernetes Three-Tier Project using Terraform, AWS EKS, ArgoCD, Prometheus, Grafana, and Jenkins

![project](https://github.com/user-attachments/assets/ef7a0f8c-deea-4453-8897-7663bf4bf44e)

## Project Introduction:
Welcome to the End-to-End DevSecOps Kubernetes Project guide! In this comprehensive project, we will walk through the process of setting up a robust Three-Tier architecture on AWS using Kubernetes, DevOps best practices, and security measures. This project aims to provide hands-on experience in deploying, securing, and monitoring a scalable application environment.

### Before we dive into these amazing projects, here is github link where you can use Terraform to provision a Production EKS Cluster.
```bash
https://github.com/Damir94/Production-Ready-EKS-Clusters-with-Terraform-and-GitHub-Actions
```

## Project Overview:
In this project, we will cover the following key aspects:

1. IAM User Setup: Create an IAM user on AWS with the necessary permissions to facilitate deployment and management activities.
2. Infrastructure as Code (IaC): Use Terraform and AWS CLI to set up the Jenkins server (EC2 instance) on AWS.
3. Jenkins Server Configuration: Install and configure essential tools on the Jenkins server, including Jenkins itself, Docker, Sonarqube, Terraform, Kubectl, AWS CLI, and Trivy.
4. EKS Cluster Deployment: Utilise eksctl commands to create an Amazon EKS cluster, a managed Kubernetes service on AWS.
5. Load Balancer Configuration: Configure AWS Application Load Balancer (ALB) for the EKS cluster.
6. Amazon ECR Repositories: Create private repositories for both frontend and backend Docker images on Amazon Elastic Container Registry (ECR).
7. ArgoCD Installation: Install and set up ArgoCD for continuous delivery and GitOps.
8. Sonarqube Integration: Integrate Sonarqube for code quality analysis in the DevSecOps pipeline.
9. Jenkins Pipelines: Create Jenkins pipelines for deploying backend and frontend code to the EKS cluster.
10. Monitoring Setup: Implement monitoring for the EKS cluster using Helm, Prometheus, and Grafana.
11. ArgoCD Application Deployment: Use ArgoCD to deploy the Three-Tier application, including database, backend, frontend, and ingress components.
12. DNS Configuration: Configure DNS settings to make the application accessible via custom subdomains.
13. Data Persistence: Implement persistent volumes and persistent volume claims for database pods to ensure data persistence.
14. Conclusion and Monitoring: Conclude the project by summarising key achievements and monitoring the EKS cluster’s performance using Grafana.

## Prerequisites:
Before starting the project, ensure you have the following prerequisites:
  - An AWS account with the necessary permissions to create resources.
  - Terraform and AWS CLI are installed on your local machine.
  - Basic familiarity with Kubernetes, Docker, Jenkins, and DevOps principles.

## Step 1: We need to create an IAM user and generate the AWS Access key
Create a new IAM User on AWS and give it AdministratorAccess for testing purposes (not recommended for your organisation's Projects)

Go to the AWS IAM Service and click on Users.

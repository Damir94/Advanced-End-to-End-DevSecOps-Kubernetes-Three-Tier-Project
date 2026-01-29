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

<img width="1373" height="433" alt="Screenshot 2026-01-27 at 12 20 27 PM" src="https://github.com/user-attachments/assets/79c45545-9e3a-40be-bea5-fb7b57eac9b2" />

Click on Create user

<img width="1502" height="372" alt="Screenshot 2026-01-27 at 12 21 20 PM" src="https://github.com/user-attachments/assets/e956ec1a-5209-4f4e-acc5-9e4a51c18255" />
Provide the name to your user and click on Next.

<img width="1020" height="147" alt="Screenshot 2026-01-27 at 12 22 05 PM" src="https://github.com/user-attachments/assets/0c8d53e9-e51e-441e-8ef3-0ffa979f36d9" />

Select the Attach policies directly option and search for AdministratorAccess, then select it.

<img width="1497" height="447" alt="Screenshot 2026-01-27 at 12 23 16 PM" src="https://github.com/user-attachments/assets/aa6652d7-0f99-44d6-8549-144b9b3c5f9b" />

Click on Create user

<img width="1539" height="678" alt="Screenshot 2026-01-27 at 12 24 10 PM" src="https://github.com/user-attachments/assets/00fc6650-4d81-48c6-81ae-3470d7bde26e" />

Now, select your created user, then click on Security credentials and generate an access key by clicking on Create access key.
Press enter or click to view image in full size

<img width="1549" height="257" alt="Screenshot 2026-01-27 at 12 26 42 PM" src="https://github.com/user-attachments/assets/77e6b6e3-0624-405b-ae2b-7914f849455b" />

Select the Command Line Interface (CLI), then select the check mark for the confirmation and click on Next.

<img width="787" height="130" alt="Screenshot 2026-01-27 at 12 27 10 PM" src="https://github.com/user-attachments/assets/354f1432-e80c-4cb3-8cea-6c0749971b43" />

Provide the Description and click on the Create access key.

<img width="838" height="160" alt="Screenshot 2026-01-27 at 12 27 57 PM" src="https://github.com/user-attachments/assets/c765d123-1459-4fb5-a0f7-8f870ae6355a" />

Here, you will see that you got the credentials, and you can also download the CSV file for the future.

## Step 2: We will install Terraform & AWS CLI to deploy our Jenkins Server(EC2) on AWS.
Install & Configure Terraform and AWS CLI on your local machine to create a Jenkins Server on AWS Cloud

### Terraform Installation Script
```bash
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg - dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update
sudo apt install terraform -y
```
### AWSCLI Installation Script
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip -y
unzip awscliv2.zip
sudo ./aws/install
```
Now, configure both the tools

### Configure Terraform
Edit the file /etc/environment using the below command, add the highlighted lines and add your keys in the blur space.
```bash
sudo vim /etc/environment
```
After making the changes, restart your machine to reflect the changes to your environment variables.

### Configure AWS CLI
Run the below command, and add your keys
```bash
aws configure
```

### Step 3: Deploy the Jenkins Server(EC2) using Terraform
Clone the Git repository
```bash
https://github.com/AmanPathak-DevOps/End-to-End-Kubernetes-Three-Tier-DevSecOps-Project
```
Navigate to the Jenkins-Server-TF

Do some modifications to the backend.tf file, such as changing the bucket name and dynamodb table(make sure you have created both manually on AWS Cloud).

Now, you have to replace the PEM file name as you have some other name for your PEM file. To provide the PEM file name that is already created on AWS

Initialise the backend by running the command below
```bash
terraform init
```
Run the command below to check the syntax error
```bash
terraform validate
```
Run the below command to get the blueprint of what kind of AWS services will be created.
```bash
terraform plan -var-file=variables.tfvars
```
Now, run the below command to create the infrastructure on AWS Cloud, which will take 3 to 4 minutes maximum
```bash
terraform apply -var-file=variables.tfvars --auto-approve
```
Now, connect to your Jenkins server by clicking on Connect.

Copy the SSH command and paste it on your local machine.

### Step 4: Configure the Jenkins
Now, we logged into our Jenkins server.

We have installed some services such as Jenkins, Docker, Sonarqube, Terraform, Kubectl, AWS CLI, and Trivy.
Let’s validate whether all our installed or not.
```bash
jenkins --version
docker --version
docker ps
terraform --version
kubectl version
aws --version
trivy --version
eksctl --version
```

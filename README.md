## Advanced End-to-End DevSecOps Kubernetes Three-Tier Project using Terraform, AWS EKS, ArgoCD, Prometheus, Grafana, and Jenkins

![project](https://github.com/user-attachments/assets/ef7a0f8c-deea-4453-8897-7663bf4bf44e)

## Project Introduction:
Welcome to the End-to-End DevSecOps Kubernetes Project guide! In this comprehensive project, we will walk through the process of setting up a robust Three-Tier architecture on AWS using Kubernetes, DevOps best practices, and security measures. This project aims to provide hands-on experience in deploying, securing, and monitoring a scalable application environment.

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

- Go to the AWS IAM Service and click on Users.

<img width="1373" height="433" alt="Screenshot 2026-01-27 at 12 20 27 PM" src="https://github.com/user-attachments/assets/79c45545-9e3a-40be-bea5-fb7b57eac9b2" />

- Click on Create user

<img width="1502" height="372" alt="Screenshot 2026-01-27 at 12 21 20 PM" src="https://github.com/user-attachments/assets/e956ec1a-5209-4f4e-acc5-9e4a51c18255" />

- Provide the name to your user and click on Next.

<img width="1020" height="147" alt="Screenshot 2026-01-27 at 12 22 05 PM" src="https://github.com/user-attachments/assets/0c8d53e9-e51e-441e-8ef3-0ffa979f36d9" />

- Select the Attach policies directly option and search for AdministratorAccess, then select it.

<img width="1497" height="447" alt="Screenshot 2026-01-27 at 12 23 16 PM" src="https://github.com/user-attachments/assets/aa6652d7-0f99-44d6-8549-144b9b3c5f9b" />

- Click on Create user

<img width="1539" height="678" alt="Screenshot 2026-01-27 at 12 24 10 PM" src="https://github.com/user-attachments/assets/00fc6650-4d81-48c6-81ae-3470d7bde26e" />

- Now, select your created user, then click on Security credentials and generate an access key by clicking on Create access key.
- Press enter or click to view image in full size

<img width="1549" height="257" alt="Screenshot 2026-01-27 at 12 26 42 PM" src="https://github.com/user-attachments/assets/77e6b6e3-0624-405b-ae2b-7914f849455b" />

- Select the Command Line Interface (CLI), then select the check mark for the confirmation and click on Next.

<img width="787" height="130" alt="Screenshot 2026-01-27 at 12 27 10 PM" src="https://github.com/user-attachments/assets/354f1432-e80c-4cb3-8cea-6c0749971b43" />

- Provide the Description and click on the Create access key.

<img width="838" height="160" alt="Screenshot 2026-01-27 at 12 27 57 PM" src="https://github.com/user-attachments/assets/c765d123-1459-4fb5-a0f7-8f870ae6355a" />

- Here, you will see that you got the credentials, and you can also download the CSV file for the future.

## Step 2: We will create our Jenkins Server(EC2) on AWS.
1. Log in to AWS
  - Go to AWS Management Console
  - Search for EC2
  - Click EC2 → Instances → Launch instance

2. Name your instance
  - Example: Jenkins-server

3. Choose an AMI (OS)
  - Pick an operating system: Ubuntu Server 22.04

4. Choose Instance Type
  - Select t2.2xlarge

5. Create or Select a Key Pair
  - This is required to SSH into EC2.
  - Click Create new key pair
  - Name it (example: ec2-key)
  - Download the .pem file
  - Save it safely — you can’t download it again.

6. Network Settings (Important)
  - Click Edit under Network settings:
  - VPC: Default
  - Subnet: Any
  - Auto-assign public IP: Enable
  - Security Group:
      - Allow SSH (22) → Source: My IP
      - Allow HTTP (80) if you want a web server
      - Allow Custom TCP (8080) port for Jenkins server
      - Alow Custom TCP (9000) port Sonarqube
7. Storage
  - Make it 30G

8. Launch Instance 
  - Click Launch instance
  - Your EC2 will be running in ~30–60 seconds.

<img width="1555" height="257" alt="Screenshot 2026-02-07 at 12 23 28 PM" src="https://github.com/user-attachments/assets/c550c483-b895-4c04-92fb-b1dec90fe37a" />

### Create an IAM Role
  - Go to IAM → Roles → Create role
  - Trusted entity:
      - Select AWS service
      - Choose EC2
  - Click Next
  - Attach Admin Access
      - Search and select: AdministratorAccess
  - Click Next 
  - Name the Role
  - Create the role

<img width="1540" height="355" alt="Screenshot 2026-02-07 at 12 56 46 PM" src="https://github.com/user-attachments/assets/1d963bee-c921-4f6e-9250-e7347eb1378d" />

### Attach Role to EC2 Instance
  - Go to EC2 → Instances
  - Select your instance
  - Actions → Security → Modify IAM role
  - Choose EC2-Admin-Role
  - Save
Your EC2 instance now has full admin access to AWS

<img width="503" height="235" alt="Screenshot 2026-02-08 at 1 21 10 PM" src="https://github.com/user-attachments/assets/703dc76c-2845-4598-b14b-1137ef52469b" />


### Jenkins Installation Guide (Ubuntu)
Step 1: Update your system
```bash
sudo apt update && sudo apt upgrade -y
```
Step 2: Install Java (Jenkins needs Java)
Jenkins works best with OpenJDK 17 now.
```bash
sudo apt install openjdk-17-jdk -y
```
Verify
```bash
java -version
```
<img width="915" height="140" alt="Screenshot 2026-02-08 at 1 23 46 PM" src="https://github.com/user-attachments/assets/53d0be19-9cb4-4a7e-84b4-1171bbcd9875" />

Step 3: Add Jenkins official repository & key
```bash
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
```
Step 4: Install Jenkins
```bash
sudo apt update
sudo apt install jenkins -y
```
Step 5: Start & enable Jenkins
```bash
sudo systemctl start jenkins
sudo systemctl enable jenkins
```
Check status:
```bash
sudo systemctl status jenkins
```
<img width="1211" height="219" alt="Screenshot 2026-02-08 at 2 04 06 PM" src="https://github.com/user-attachments/assets/b3335cc5-1a8b-4294-827e-26fa4a67541b" />

### Docker Installation Guide (Ubuntu)

Step 1: Update the Package Index
```bash
sudo apt update
```
Step 2: Install Required Dependencies
```bash
sudo apt install -y ca-certificates curl
```
Step 3: Add Docker’s Official GPG Key
```bash
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```
Step 4: Add the Docker Repository
```bash
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/ubuntu
Suites: $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF
```
```bash
sudo apt update
```
Step 5: Install Docker Engine
```bash
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
Step 6: Verify Docker Installation
```bash
sudo systemctl start docker
sudo systemctl status docker
sudo docker run hello-world
```
<img width="1032" height="278" alt="image" src="https://github.com/user-attachments/assets/a27be5d0-3098-4449-8542-700dbfec428f" />

Step 7: Run Docker Without sudo (Optional but Recommended)
```bash
sudo usermod -aG docker ubuntu
```
Important: Log out and log back in for the group changes to take effect. After re-login, you should be able to run Docker commands without sudo.

## Run Docker Container of Sonarqube
```bash
docker run -d  --name sonar -p 9000:9000 sonarqube:lts-community
```

### AWS CLI is installed
Step 1: Update your system
```bash
sudp apt update
```
Step 2: Download AWS CLI
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
```
Step 3: Unzip the installer
```bash
unzip awscliv2.zip
```
Step 4: Install AWS CLI
```bash
sudo ./aws/install
```
Step 5: Verify Installation
```bash
aws --version
```
### eksctl is installed: eksctl version
Step 1: Install
```bash
sudo apt update && sudo apt install -y curl
```
Step 2: Download the latest eksctl binary
```bash
curl -sLO https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_Linux_amd64.tar.gz
```
Step 3: Extract the binary
```bash
tar -xzf eksctl_Linux_amd64.tar.gz
```
Step 4: Move it to a system path
```bash
sudo mv eksctl /usr/local/bin
```
Step 5: Verify installation
```bash
eksctl version
```

### Kubernetes CLI (kubectl) Installation
Step 1: Download kubectl Binary and Checksum
```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
```
Step 2: Step 2: Install kubectl
```bash
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```
Step 3: Verify Installation
```bash
kubectl version --client
```
<img width="585" height="98" alt="Screenshot 2026-01-13 at 11 38 09 AM" src="https://github.com/user-attachments/assets/b4035a28-99db-48fa-aabc-9e3e890c9354" />

### Terraform Installation
Step 1: Install Required Packages
```bash
sudo apt-get update && sudo apt-get install -y gnupg software-properties-common
```
Step 2: Add HashiCorp GPG Key
```bash
wget -O- https://apt.releases.hashicorp.com/gpg | \
  gpg --dearmor | \
  sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg > /dev/null
```
Verify the key fingerprint:
```bash
gpg --no-default-keyring \
  --keyring /usr/share/keyrings/hashicorp-archive-keyring.gpg \
  --fingerprint
```
Step 3: Add HashiCorp Repository
```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(grep -oP '(?<=UBUNTU_CODENAME=).*' /etc/os-release || lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
```
Update package index and install Terraform:
```bash
sudo apt update
sudo apt-get install -y terraform
```
Verify Terraform installation:
```bash
terraform --version
```
<img width="525" height="96" alt="image" src="https://github.com/user-attachments/assets/ad84684d-6a60-43e6-90ac-bdbbdc7fe069" />

### Install Trivy using official repo
Step 1: Update system
```bash
sudo apt update
```
Step 2: Install required packages
```bash
sudo apt install -y wget apt-transport-https gnupg lsb-release
```
Step 3: Add Trivy GPG key
```bash
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
```
Step 4: Add Trivy repository
```bash
echo deb https://aquasecurity.github.io/trivy-repo/deb \
$(lsb_release -sc) main | sudo tee /etc/apt/sources.list.d/trivy.list
```
Step 5: Install Trivy
```bash
sudo apt update
sudo apt install -y trivy
```
Step 6: Verify installation
```bash
trivy --version
```

### helm is installed
Step 1: Update system
```bash
sudo apt update
```
Step 2: Download and install Helm
```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```
Step 3: Verify installation
```bash
helm version
```

### Step 3: Configure the Jenkins
- We logged into our Jenkins server.

- Now, we have to configure Jenkins. So, copy the public IP of your Jenkins Server and paste it into your favourite browser on port 8080.
<img width="833" height="779" alt="Screenshot 2026-01-30 at 10 57 33 AM" src="https://github.com/user-attachments/assets/36e715ea-1d79-4b33-afe5-79e01e6f58d3" />

- Click on Install suggested plugins
<img width="835" height="389" alt="Screenshot 2026-01-30 at 10 58 25 AM" src="https://github.com/user-attachments/assets/85a662f2-9dba-4344-99b3-8db85911a033" />

- The plugins will be installed
<img width="836" height="213" alt="Screenshot 2026-01-30 at 10 59 00 AM" src="https://github.com/user-attachments/assets/4910a655-e3cb-4e58-a27c-e05c693b2cb9" />

- After installing the plugins, continue as admin

<img width="831" height="770" alt="Screenshot 2026-01-30 at 10 59 40 AM" src="https://github.com/user-attachments/assets/71931e51-7b21-4875-8a60-8dbe9e04dc50" />
 
- Click on Save and Finish
- Click on Start using Jenkins

- The Jenkins Dashboard will look like the snippet below

<img width="1594" height="541" alt="Screenshot 2026-01-30 at 11 00 58 AM" src="https://github.com/user-attachments/assets/b0da7b80-71bd-4781-91a7-47c5534642e9" />

### Step 5: We will deploy the EKS Cluster using the eksctl commands
Now, go back to your Jenkins Server terminal and configure the AWS.
```bash
aws configure
```
  - Go to Manage Jenkins
  - Click on Plugins
<img width="1577" height="381" alt="Screenshot 2026-01-30 at 11 03 23 AM" src="https://github.com/user-attachments/assets/dd08a6cb-a656-47f5-af36-44a438ebc24f" />

Select the Available plugins, install the following plugins and click on Install
  - AWS Credentials
  - Pipeline: AWS Steps
<img width="1279" height="291" alt="Screenshot 2026-01-30 at 11 04 16 AM" src="https://github.com/user-attachments/assets/6419415c-b9fc-4210-8c04-92f630af3a05" />

- Once both plugins are installed, restart your Jenkins service by checking the Restart Jenkins option.
- Log in to your Jenkins Server Again

#### Now, we have to set our AWS credentials on Jenkins
- Go to Manage Plugins and click on Credentials
<img width="1577" height="559" alt="Screenshot 2026-01-30 at 11 06 04 AM" src="https://github.com/user-attachments/assets/c7b1c7e4-6e05-499a-9f71-4680398699d8" />

- Click on global.
<img width="1357" height="270" alt="Screenshot 2026-01-30 at 11 06 34 AM" src="https://github.com/user-attachments/assets/815423dc-d74c-48e7-87f1-d6e627b6a0a6" />

- Select AWS Credentials as Kind and add the ID same as shown in the below snippet, except for your AWS Access Key & Secret Access key, and click on Create.
<img width="720" height="342" alt="image" src="https://github.com/user-attachments/assets/1abca0c0-b2d1-43a3-a342-9710d2d3bad5" />

- The Credentials will look like the snippet below.
<img width="1600" height="288" alt="image" src="https://github.com/user-attachments/assets/e7bed2eb-323b-4d8e-92d9-76599fbf27c7" />

- Now, we need to add GitHub credentials as well because currently, my repository is Private.
- This thing, I am performing this because in Industry Projects, your repository will be private.
- So, add the username and personal access token of your GitHub account.
<img width="1600" height="680" alt="image" src="https://github.com/user-attachments/assets/cff93a30-ac62-428f-8d1e-73341ea41bcf" />

- Both credentials will look like this.
<img width="720" height="149" alt="image" src="https://github.com/user-attachments/assets/c891e209-4329-46f7-ac91-5023f1e62b5f" />

### Install Terraform Plugin via Jenkins UI
Step 1: Open Jenkins Dashboard
```nginx
Manage Jenkins → Manage Plugins
```
Step 2: Install Plugin
  - Go to Available plugins
  - Search: Terraform
  - Install: Terraform Plugin (by HashiCorp)
  - Check “Restart Jenkins after install”

Step 3: Configure Terraform Installation
After restart:
```nginx
Manage Jenkins → Global Tool Configuration
```

Step 4: Use system-installed Terraform
  - If Terraform is already installed on the server:
  - Uncheck Install automatically
  - Set path:
```bash
/usr/bin/terraform
```

### We will deploy our EKS Cluster with help of Jenkins file 
Big Picture (what we’re building)
```csharp
GitHub Repo (Terraform EKS code)
        ↓
Jenkins Pipeline Job
        ↓
Terraform init → plan → apply
        ↓
AWS EKS Cluster Created
```

Step 1: Jenkinsfile (Pipeline Code)
  - Create a file named Jenkinsfile in your repo:
```groovy
properties([
    parameters([
        string(
            defaultValue: 'dev',
            name: 'Environment'
        ),
        choice(
            choices: ['plan', 'apply', 'destroy'], 
            name: 'Terraform_Action'
        )])
])
pipeline {
    agent any
    stages {
        stage('Preparing') {
            steps {
                sh 'echo Preparing'
            }
        }
        stage('Git Pulling') {
            steps {
                git branch: 'master', url: 'https://github.com/AmanPathak-DevOps/EKS-Terraform-GitHub-Actions.git'
            }
        }
        stage('Init') {
            steps {
                withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                sh 'terraform -chdir=eks/ init'
                }
            }
        }
        stage('Validate') {
            steps {
                withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                sh 'terraform -chdir=eks/ validate'
                }
            }
        }
        stage('Action') {
            steps {
                withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                    script {    
                        if (params.Terraform_Action == 'plan') {
                            sh "terraform -chdir=eks/ plan -var-file=${params.Environment}.tfvars"
                        }   else if (params.Terraform_Action == 'apply') {
                            sh "terraform -chdir=eks/ apply -var-file=${params.Environment}.tfvars -auto-approve"
                        }   else if (params.Terraform_Action == 'destroy') {
                            sh "terraform -chdir=eks/ destroy -var-file=${params.Environment}.tfvars -auto-approve"
                        } else {
                            error "Invalid value for Terraform_Action: ${params.Terraform_Action}"
                        }
                    }
                }
            }
        }
    }
}
```

Step 2: Prepare Your GitHub Repo
Your repo should look something like this:
```css
eks/
├── main.tf
├── variables.tf
├── outputs.tf
├── provider.tf
├── vpc.tf
├── eks.tf
Jenkinsfile
```
Step 3: Create Jenkins Pipeline Job
  - Jenkins Dashboard → New Item
  - Name: deploy-eks-cluster
  - Select: Pipeline
  - Click OK

Step 4: Configure Pipeline Job
  - Under Pipeline section:
  - Definition: Pipeline script from SCM
  - SCM: Git
  - Repository URL: Your GitHub repo URL
  - Branch: */main
  - Script Path: Jenkinsfile
  - Save

Step 5: Run the Pipeline
  - Open the job → Build Now
  - Watch Console Output
  - You should see:
      - Terraform init
      - Terraform plan
      - Terraform apply
### A jump server (or bastion host)
You need a jump server to reach an EKS cluster in a private VPC because private subnets and private endpoints are not accessible from the public internet, and the jump server acts as a secure gateway to connect to your cluster.

### Why EKS clusters may need a jump server
  - When you create an EKS cluster inside a VPC, you usually put the worker nodes (EC2 instances) or private API endpoints in private subnets for security.
  - Private subnets: Not directly accessible from the internet.
  - EKS API endpoint: Can be private (accessible only inside the VPC) or public (accessible over the internet).
  - If the EKS API endpoint is private, you cannot reach it from your laptop or home network directly.

### This is where the jump server comes in:
  - You SSH into the jump server (which is publicly accessible).
  - From there, you can access the private EKS API endpoint or worker nodes.
  - It’s essentially your “doorway” into the private network.

1. Log in to AWS
  - Go to AWS Management Console
  - Search for EC2
  - Click EC2 → Instances → Launch instance

2. Name your instance
  - Example: Jump-server

3. Choose an AMI (OS)
  - Pick an operating system: Ubuntu Server 22.04

4. Choose Instance Type
  - Select t2.medium

5. Create or Select a Key Pair
  - This is required to SSH into EC2.
  - Click Create new key pair
  - Name it (example: ec2-key)
  - Download the .pem file
  - Save it safely — you can’t download it again.

6. Network Settings (Important)
  - Click Edit under Network settings:
  - VPC: Choose EKS cluster VPC
  - Subnet: Any public subnet from EKS cluster VPC
  - Auto-assign public IP: Enable
  - Security Group: None
      
7. Storage
  - Make it 30G

8. Add adminstrator access to the jump server through IAM role in the IAM instance profile

9. Launch Instance 
  - Click Launch instance
  - Your EC2 will be running in ~30–60 seconds.
10. We need to install AWS CLI, kubectl, helm on the jump server
#### Installing AWS CLI
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip -y
unzip awscliv2.zip
sudo ./aws/install
```
#### Installing Kubectl
```bash
sudo apt update
sudo apt install curl -y
sudo curl -LO "https://dl.k8s.io/release/v1.28.4/bin/linux/amd64/kubectl"
sudo chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client
```
#### Installing Helm
```bash
sudo snap install helm --classic
```
####  Installing eksctl
```bash
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```
### Verify your jump server can reach the EKS cluster
Step 1: Provide  aws credentials and run this on jump server:
```bash
aws configure
```
You’ll be asked for:
  - AWS Access Key ID
  - AWS Secret Access Key
  - Default region (e.g., us-east-1)
  - Default output format (optional, e.g., json)
Step 2: After configuring, you can run:
```bash
aws eks --region <region> update-kubeconfig --name <cluster-name>
kubectl get nodes
```
Security note: Don’t leave long-lived access keys on a jump server if multiple people have access. Using an instance IAM role is safer.

### Installing AWS ALB Ingress Controller on EKS
This guide explains how to install the AWS ALB (AWS Load Balancer) Ingress Controller on an EKS cluster, along with the required IAM setup.

Step 1: Set the cluster name:
```bash
export cluster_name=demo-cluster
```
Step 2: Setup OIDC Connector
```bash
oidc_id=$(aws eks describe-cluster --name $cluster_name --query "cluster.identity.oidc.issuer" --output text | cut -d '/' -f 5)
```
Step 3: Check if OIDC provider already exists
```bash
aws iam list-open-id-connect-providers | grep $oidc_id | cut -d "/" -f4
```
Step 4: If OIDC is NOT configured, run this
```bash
eksctl utils associate-iam-oidc-provider --cluster $cluster_name --approve
```
Step 5: Download IAM Policy for ALB Controller
```bash
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.11.0/docs/install/iam_policy.json
```
Step 6: Create the IAM Policy in AWS
```bash
aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json
```
Step 7: Create IAM Role + Kubernetes Service Account
```bash
eksctl create iamserviceaccount \
  --cluster=<your-cluster-name> \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
--attach-policy-arn=arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve --region=us-east-1 --override-existing-serviceaccounts
```
Step 8: Verify Service Account is create by running:
```bash
kubectl get sa -n kube-system
```
If there is no Service Account created, go to the AWS CloudFormation and delete service account

<img width="1217" height="425" alt="Screenshot 2026-02-09 at 11 37 24 AM" src="https://github.com/user-attachments/assets/f1384b43-d7d3-41d0-ace7-ce14c35168de" />

Step 9: We will run same command again:
```bash
eksctl create iamserviceaccount \
  --cluster=<your-cluster-name> \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
--attach-policy-arn=arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve --region=us-east-1 --override-existing-serviceaccounts
```

<img width="1204" height="463" alt="Screenshot 2026-02-09 at 11 41 38 AM" src="https://github.com/user-attachments/assets/c5d3c02d-3ea1-4ffd-8f4e-d1f1c86a38e8" />

Step 10:  Verify Service Account is create by running:
```bash
kubectl get sa -n kube-system
```

<img width="590" height="142" alt="Screenshot 2026-02-09 at 11 43 55 AM" src="https://github.com/user-attachments/assets/a81beca2-39df-43bd-8945-ddac31344861" />

Step 11: Deploy the ALB Controller using Helm
```bash
helm repo add eks https://aws.github.io/eks-charts
helm repo update eks
helm repo list
```
<img width="826" height="231" alt="Screenshot 2026-02-09 at 11 46 09 AM" src="https://github.com/user-attachments/assets/0eb345b2-69f8-4fca-9a47-ea4afda96c6a" />

Step 12: Install the ALB Controller
```bash
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \            
  -n kube-system \
  --set clusterName=<your-cluster-name> \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=<region> \
  --set vpcId=<your-vpc-id>
```
Step 13: Verify the Controller is Running
```bash
kubectl get pods -n kube-system
kubectl get deployment -n kube-system aws-load-balancer-controller
```
### For this setup, Argo CD is installed using plain Kubernetes manifests, as recommended for getting started.
Install Argo CD
Step 1: Create the Argo CD Namespace
```bash
kubectl create namespace argocd
```
Step 2: Apply the Argo CD Manifests
```bash
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
Step 3: Verify Installation
```bash
kubectl get pods -n argocd
```
Step 4: Check Services:
```bash
kubectl get svc -n argocd
```
Step 5: Expose Argo CD UI
```bash
kubectl edit svc argocd-server -n argocd
```
Change: type: ClusterIP to type: LoadBalancer

<img width="420" height="146" alt="image" src="https://github.com/user-attachments/assets/27a93ad8-5e58-42e5-8373-dcc6078acab7" />

Step 6: Then verify and loadbalancer dns name:
```bash
kubectl get svc -n argocd
```
Step 7: Login to Argo CD
```bash
kubectl get secrets -n argocd
```
Look for: argocd-initial-admin-secret

Step 8: Decode the Password
```bash
kubectl get secret argocd-initial-admin-secret \
  -n argocd \
  -o jsonpath="{.data.password}" | base64 --decode
```

### we will configure SonarQube for our DevSecOps Pipeline
Step 1: To do that, copy your Jenkins Server public IP and paste it into your favourite browser with a 9000 port
```bash
curl ifconfig.me
```
Step 2: The username and password will be admin and Click on Log In.

<img width="720" height="127" alt="image" src="https://github.com/user-attachments/assets/53511e88-715f-46b7-877e-900f500184a1" />

Step 3: Update the password

<img width="720" height="242" alt="image" src="https://github.com/user-attachments/assets/404fbaab-c67f-4d85-8643-766be7b5d126" />

Step 4: Click on Administration, then Security, and select Users

<img width="720" height="124" alt="image" src="https://github.com/user-attachments/assets/6b47578f-da83-4d39-8a15-45c8d7928854" />

Step 5: Click on Update tokens

<img width="720" height="176" alt="image" src="https://github.com/user-attachments/assets/53b07ccc-1f4e-4e44-845a-3af8dec297d6" />

Step 6: Click on Generate

<img width="720" height="176" alt="image" src="https://github.com/user-attachments/assets/3b258a6d-cf2f-47aa-ae4f-a8ae1651c7e6" />

Step 7: Copy the token, keep it somewhere safe and click on Done.

<img width="720" height="215" alt="image" src="https://github.com/user-attachments/assets/c9f312a1-eb5e-4f75-b35c-b92459debe7e" />

Now, we have to configure webhooks for quality checks.

Step 8: Click on Administration, then Configuration, and select Webhooks

<img width="720" height="93" alt="image" src="https://github.com/user-attachments/assets/c2af05d0-86bd-4c2a-88ef-2256156bbda6" />

Step 9: Click on Create

<img width="720" height="123" alt="image" src="https://github.com/user-attachments/assets/7dc0dbb0-5781-4a56-a460-b3046354833e" />

Step 10: Provide the name of your project and in the URL, provide the Jenkins server public IP with port 8080, add sonarqube-webhook in the suffix, and click on Create.
```bash
http://<jenkins-server-public-ip>:8080/sonarqube-webhook/
```

<img width="720" height="290" alt="image" src="https://github.com/user-attachments/assets/6d04abfe-ca0b-433b-8ea2-1812c5661397" />

Step 11: Here, you can see the webhook.

<img width="1600" height="326" alt="image" src="https://github.com/user-attachments/assets/3a0e64d4-82c2-489f-a0ce-2021b6c0875c" />

Step 11: We will create a Project for the frontend code. Click on Manually.

<img width="720" height="260" alt="image" src="https://github.com/user-attachments/assets/d492103c-e097-4d6e-af46-a5b93c80c471" />

Step 12: Provide the display name to your Project and click on Setup

<img width="1600" height="480" alt="image" src="https://github.com/user-attachments/assets/b2ebf24c-9fe1-4aea-b3e1-9a4f8b407d8d" />

Step 13: Click on Locally.

<img width="720" height="273" alt="image" src="https://github.com/user-attachments/assets/00d1cbaa-6cca-4d08-ac78-b13e4acdddff" />

Step 14: Select the Use existing token and click on Continue.

<img width="720" height="203" alt="image" src="https://github.com/user-attachments/assets/fc461202-2475-4191-911b-a01e65e3a28d" />

Step 15: Select Other and Linux as OS.

<img width="720" height="347" alt="image" src="https://github.com/user-attachments/assets/90b26a17-3f1c-4b5a-8206-6d943729b86a" />

After performing the above steps, you will get the command, which you can see in the snippet above.

Now, use the command in the Jenkins Frontend Pipeline where Code Quality Analysis will be performed.

Step 16: We will create a Project for the backend code. Click on Create Project.

<img width="720" height="135" alt="image" src="https://github.com/user-attachments/assets/0b4399cd-5163-4c41-b5a4-110c460eebc0" />

Step 17: Provide the name of your project and click on Set up.

<img width="720" height="215" alt="image" src="https://github.com/user-attachments/assets/09824f6c-7185-4634-98d7-010a7f08726c" />

Step 18: Click on Locally.

<img width="720" height="269" alt="image" src="https://github.com/user-attachments/assets/f79e9ace-9ae0-4fbe-9f94-49e49b431ca6" />

Step 19: Select the Use existing token and click on Continue.

<img width="720" height="204" alt="image" src="https://github.com/user-attachments/assets/8b132c92-4b17-477d-88ec-6ac0ef3780bd" />

Step 20: Select Other and Linux as OS.

<img width="720" height="347" alt="image" src="https://github.com/user-attachments/assets/5b40135f-eeb2-4dc4-8f05-82c676f6cb4f" />

After performing the above steps, you will get the command, which you can see in the snippet above.

Use the command in the Jenkins Backend Pipeline where Code Quality Analysis will be performed.

### We will store Sonar credentials.

Step 1: Go to Dashboard -> Manage Jenkins -> Credentials

Step 2: Select the kind as Secret text, paste your token in Secret and keep other things as it is.

Step 3: Click on Create

<img width="1600" height="559" alt="image" src="https://github.com/user-attachments/assets/66bbd523-1673-4366-8a11-eb3a9d3aa097" />

### We will store the GitHub Personal access token to push the deployment file, which will be modified in the pipeline itself for the ECR image.

Step 1: Add GitHub credentials

Step 2: Select the kind as Secret text and paste your GitHub Personal access token(not password) in Secret, and keep other things as it is.

Step 3: Click on Create

Note: If you haven’t generated your token, you generate it first, then paste it into the Jenkins

<img width="1600" height="559" alt="image" src="https://github.com/user-attachments/assets/ac36f771-b49d-4a8e-bdb8-ca54aeefd1c5" />

### According to our Pipeline, we will add an AWS Account ID in the Jenkins credentials because of the ECR repo URI.

Step 1: Select the kind as Secret text, paste your AWS Account ID in Secret and keep other things as it is.

Step 2: Click on Create

<img width="720" height="252" alt="image" src="https://github.com/user-attachments/assets/a6ff10f7-48e6-4940-a6b9-bb7a4b5a3aab" />

### We will create Amazon ECR Private Repositories for both Tiers (Frontend & Backend)

Step 1: Click on Create repository

<img width="720" height="150" alt="image" src="https://github.com/user-attachments/assets/e3db6694-3787-4b92-a69a-106d304278f5" />

Step 2: Select the Private option to provide the repository and click on Save.

<img width="720" height="150" alt="image" src="https://github.com/user-attachments/assets/f45f99fa-8482-40db-80b2-f4b7cd6624db" />

Step 3: Do the same for the backend repository and click on Save

<img width="720" height="150" alt="image" src="https://github.com/user-attachments/assets/2b531e3a-9e2b-466f-8154-3043e14d6743" />

Step 4: we have set up our ECR Private Repository

<img width="720" height="150" alt="image" src="https://github.com/user-attachments/assets/f547c539-d518-4e8b-bdb4-cd84227d53ef" />

### We will provide our ECR image name for the frontend, which is frontend only.

Step 1: Select the kind as Secret text, paste your frontend repo name in Secret and keep other things as it is.

Step 2: Click on Create

<img width="720" height="252" alt="image" src="https://github.com/user-attachments/assets/5a046e33-81eb-4343-89bd-94e2accd0f42" />

### We will provide our ECR image name for the backend, which is backend only.

Step 1: Select the kind as Secret text, paste your backend repo name in Secret, and keep other things as it is.

Step 2: Click on Create

<img width="1600" height="559" alt="image" src="https://github.com/user-attachments/assets/6b8c179c-e9b9-4490-8e61-71280e1edd24" />

### Final Snippet of all Credentials that we needed to implement this project.

<img width="720" height="252" alt="image" src="https://github.com/user-attachments/assets/8de64a75-0337-479e-8bfa-e62674bbe344" />

### Install the required plugins and configure the plugins to deploy our Three-Tier Application

Step 1: Install the following plugins by going to Dashboard -> Manage Jenkins -> Plugins -> Available Plugins
```bash
Docker
Docker Commons
Docker Pipeline
Docker API
docker-build-step
Eclipse Temurin installer
NodeJS
OWASP Dependency-Check
SonarQube Scanner
```

<img width="720" height="338" alt="image" src="https://github.com/user-attachments/assets/d4ddacb0-ffb8-4d1b-831c-017546638c3e" />

### We will configure the installed plugins.

Step 1: Go to Dashboard -> Manage Jenkins -> Tools

Step 2: We are configuring JDK

Step 3: Search for JDK and provide the configuration like the snippet below.


<img width="720" height="271" alt="image" src="https://github.com/user-attachments/assets/b3f1904d-ddf5-4ec2-8d80-82bafd020f61" />

### We will configure the SonarQube scanner

Step 1: Search for the SonarQube scanner and provide the configuration like the snippet below.

<img width="720" height="266" alt="image" src="https://github.com/user-attachments/assets/59dbb3a6-2973-4d38-86a4-e26652855729" />

### We will configure nodejs

Step 1: Search for the node and provide the configuration, like the snippet below.

<img width="720" height="369" alt="image" src="https://github.com/user-attachments/assets/67ef012c-6014-4382-8120-bc74a1c8a573" />

### We will configure the OWASP Dependency Check

Step 1: Search for Dependency-Check and provide the configuration like the snippet below.

<img width="720" height="272" alt="image" src="https://github.com/user-attachments/assets/7d96b612-b67a-4612-9d84-fd0c80db42ca" />

### we will configure the Docker

Step 1: Search for Docker and provide the configuration like the snippet below.

<img width="720" height="272" alt="image" src="https://github.com/user-attachments/assets/f379122e-1d2c-4293-93ed-b65f8e4ef7aa" />

### We will set the path for SonarQube in Jenkins

Step 1: Go to Dashboard -> Manage Jenkins -> System

Step 2: Search for SonarQube installations

Step 3: Provide the name as it is, then in the Server URL, copy the SonarQube public IP (same as Jenkins) with port 9000, select the Sonar token that we have added recently, and click on Apply & Save.

<img width="720" height="365" alt="image" src="https://github.com/user-attachments/assets/92940f17-12cb-4b12-9b66-bf2e6cf5b73a" />

### We are ready to create our Jenkins Pipeline to deploy our Backend Code.

Step 1: Go to Jenkins Dashboard

Step 2: Click on New Item

<img width="720" height="154" alt="image" src="https://github.com/user-attachments/assets/956be226-dc19-40a1-b79f-33090e2c3410" />

Step 3: Provide the name of your Pipeline and click on OK.

<img width="720" height="195" alt="image" src="https://github.com/user-attachments/assets/9a1da444-6985-4145-9726-a5a740a1658d" />

Step 4: This is the Jenkins file to deploy the Backend Code on EKS. Copy and paste it into the Jenkins

```groovy
pipeline {
    agent any 
    tools {
        nodejs 'nodejs'
    }
    environment  {
        SCANNER_HOME=tool 'sonar-scanner'
        AWS_ACCOUNT_ID = credentials('ACCOUNT_ID')
        AWS_ECR_REPO_NAME = credentials('ECR_REPO2')
        AWS_DEFAULT_REGION = 'us-east-1'
        REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/"
    }
    stages {
        stage('Cleaning Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git credentialsId: 'GITHUB', url: 'https://github.com/AmanPathak-DevOps/End-to-End-Kubernetes-Three-Tier-DevSecOps-Project.git'
            }
        }
        stage('Sonarqube Analysis') {
            steps {
                dir('Application-Code/backend') {
                    withSonarQubeEnv('sonar-server') {
                        sh ''' $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=backend \
                        -Dsonar.projectKey=backend '''
                    }
                }
            }
        }
        stage('Quality Check') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token' 
                }
            }
        }
       // stage('OWASP Dependency-Check Scan') {
      //     steps {
      //          dir('Application-Code/backend') {
      //              dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
      //              dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
      //          }
      //      }
      //  }
        stage('Trivy File Scan') {
            steps {
                dir('Application-Code/backend') {
                    sh 'trivy fs . > trivyfs.txt'
                }
            }
        }
        stage("Docker Image Build") {
            steps {
                script {
                    dir('Application-Code/backend') {
                            sh 'docker system prune -f'
                            sh 'docker container prune -f'
                            sh 'docker build -t ${AWS_ECR_REPO_NAME} .'
                    }
                }
            }
        }
        stage("ECR Image Pushing") {
            steps {
                script {
                        sh 'aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${REPOSITORY_URI}'
                        sh 'docker tag ${AWS_ECR_REPO_NAME} ${REPOSITORY_URI}${AWS_ECR_REPO_NAME}:${BUILD_NUMBER}'
                        sh 'docker push ${REPOSITORY_URI}${AWS_ECR_REPO_NAME}:${BUILD_NUMBER}'
                }
            }
        }
        stage("TRIVY Image Scan") {
            steps {
                sh 'trivy image ${REPOSITORY_URI}${AWS_ECR_REPO_NAME}:${BUILD_NUMBER} > trivyimage.txt' 
            }
        }
        stage('Checkout Code') {
            steps {
                git credentialsId: 'GITHUB', url: 'https://github.com/AmanPathak-DevOps/End-to-End-Kubernetes-Three-Tier-DevSecOps-Project.git'
            }
        }
        stage('Update Deployment file') {
            environment {
                GIT_REPO_NAME = "End-to-End-Kubernetes-Three-Tier-DevSecOps-Project"
                GIT_USER_NAME = "AmanPathak-DevOps"
            }
            steps {
                dir('Kubernetes-Manifests-file/Backend') {
                    withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                        sh '''
                            git config user.email "aman07pathak@gmail.com"
                            git config user.name "AmanPathak-DevOps"
                            BUILD_NUMBER=${BUILD_NUMBER}
                            echo $BUILD_NUMBER
                            imageTag=$(grep -oP '(?<=backend:)[^ ]+' deployment.yaml)
                            echo $imageTag
                            sed -i "s/${AWS_ECR_REPO_NAME}:${imageTag}/${AWS_ECR_REPO_NAME}:${BUILD_NUMBER}/" deployment.yaml
                            git add deployment.yaml
                            git commit -m "Update deployment Image to version \${BUILD_NUMBER}"
                            git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:master
                        '''
                    }
                }
            }
        }
    }
}
```
Step 5: Click Apply & Save. Click on the build.
- Our pipeline was successful after addressing a few common mistakes.
- Note: Do the changes in the Pipeline according to your project.

<img width="720" height="344" alt="image" src="https://github.com/user-attachments/assets/905277ca-a890-4b8e-b0f8-cc06bebada45" />

### We are ready to create our Jenkins Pipeline to deploy our Frontend Code.

Step 1: Go to Jenkins Dashboard

Step 2: Click on New Item

Step 3: Provide the name of your Pipeline and click on OK.

<img width="720" height="222" alt="image" src="https://github.com/user-attachments/assets/466a6c4e-2080-47f0-a2f6-8a5d7678f31a" />

Step 4: This is the Jenkins file to deploy the Frontend Code on EKS. Copy and paste it into the Jenkins
```groovy
pipeline {
    agent any 
    tools {
        nodejs 'nodejs'
    }
    environment  {
        SCANNER_HOME=tool 'sonar-scanner'
        AWS_ACCOUNT_ID = credentials('ACCOUNT_ID')
        AWS_ECR_REPO_NAME = credentials('ECR_REPO1')
        AWS_DEFAULT_REGION = 'us-east-1'
        REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/"
    }
    stages {
        stage('Cleaning Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git credentialsId: 'GITHUB', url: 'https://github.com/AmanPathak-DevOps/End-to-End-Kubernetes-Three-Tier-DevSecOps-Project.git'
            }
        }
        stage('Sonarqube Analysis') {
            steps {
                dir('Application-Code/frontend') {
                    withSonarQubeEnv('sonar-server') {
                        sh ''' $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=frontend \
                        -Dsonar.projectKey=frontend '''
                    }
                }
            }
        }
        stage('Quality Check') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token' 
                }
            }
        }
       // stage('OWASP Dependency-Check Scan') {
       //     steps {
       //         dir('Application-Code/frontend') {
       //             dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
       //             dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
       //         }
       //     }
       // }
        stage('Trivy File Scan') {
            steps {
                dir('Application-Code/frontend') {
                    sh 'trivy fs . > trivyfs.txt'
                }
            }
        }
        stage("Docker Image Build") {
            steps {
                script {
                    dir('Application-Code/frontend') {
                            sh 'docker system prune -f'
                            sh 'docker container prune -f'
                            sh 'docker build -t ${AWS_ECR_REPO_NAME} .'
                    }
                }
            }
        }
        stage("ECR Image Pushing") {
            steps {
                script {
                        sh 'aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${REPOSITORY_URI}'
                        sh 'docker tag ${AWS_ECR_REPO_NAME} ${REPOSITORY_URI}${AWS_ECR_REPO_NAME}:${BUILD_NUMBER}'
                        sh 'docker push ${REPOSITORY_URI}${AWS_ECR_REPO_NAME}:${BUILD_NUMBER}'
                }
            }
        }
        stage("TRIVY Image Scan") {
            steps {
                sh 'trivy image ${REPOSITORY_URI}${AWS_ECR_REPO_NAME}:${BUILD_NUMBER} > trivyimage.txt' 
            }
        }
        stage('Checkout Code') {
            steps {
                git credentialsId: 'GITHUB', url: 'https://github.com/AmanPathak-DevOps/End-to-End-Kubernetes-Three-Tier-DevSecOps-Project.git'
            }
        }
        stage('Update Deployment file') {
            environment {
                GIT_REPO_NAME = "End-to-End-Kubernetes-Three-Tier-DevSecOps-Project"
                GIT_USER_NAME = "AmanPathak-DevOps"
            }
            steps {
                dir('Kubernetes-Manifests-file/Frontend') {
                    withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                        sh '''
                            git config user.email "aman07pathak@gmail.com"
                            git config user.name "AmanPathak-DevOps"
                            BUILD_NUMBER=${BUILD_NUMBER}
                            echo $BUILD_NUMBER
                            imageTag=$(grep -oP '(?<=frontend:)[^ ]+' deployment.yaml)
                            echo $imageTag
                            sed -i "s/${AWS_ECR_REPO_NAME}:${imageTag}/${AWS_ECR_REPO_NAME}:${BUILD_NUMBER}/" deployment.yaml
                            git add deployment.yaml
                            git commit -m "Update deployment Image to version \${BUILD_NUMBER}"
                            git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:master
                        '''
                    }
                }
            }
        }
    }
}
```
Step 5: Click Apply & Save. 
- Our pipeline was successful after a few common mistakes.
- Note: Do the changes in the Pipeline according to your project.

<img width="720" height="323" alt="image" src="https://github.com/user-attachments/assets/feb81290-cc92-4c1c-99cf-3966e1aedbb0" />



### Conclusion
- In this comprehensive DevSecOps Kubernetes project, we successfully:
  - Established IAM user and Terraform for AWS setup.
  - Deployed Jenkins on AWS, configured tools, and integrated it with SonarQube.
  - Set up an EKS cluster, configured a Load Balancer, and established private ECR repositories.
  - Implemented monitoring with Helm, Prometheus, and Grafana.
  - Installed and configured ArgoCD for GitOps practices.
  - Created Jenkins pipelines for CI/CD, deploying a Three-Tier application.
  - Ensured data persistence with persistent volumes and claims.


## Advanced End-to-End DevSecOps Kubernetes Three-Tier Project using Terraform, AWS EKS, ArgoCD, Prometheus, Grafana, and Jenkins

<img width="1024" height="1536" alt="End-to-end-project-diagram" src="https://github.com/user-attachments/assets/3474c0a9-c47c-4598-8864-34161e680fe5" />

## Project Introduction:
Welcome to the End-to-End DevSecOps Kubernetes Project guide! In this comprehensive project, we will walk through the process of setting up a robust Three-Tier architecture on AWS using Kubernetes, DevOps best practices, and security measures. This project aims to provide hands-on experience in deploying, securing, and monitoring a scalable application environment.

## Prerequisites:
Before starting the project, ensure you have the following prerequisites:
  - An AWS account with the necessary permissions to create resources.
  - Terraform and AWS CLI are installed on your local machine.
  - Basic familiarity with Kubernetes, Docker, Jenkins, and DevOps principles.

### Why Are We Using Jenkins in This Project?
Because we want everything to happen automatically, not manually.
Instead of:
  - Writing code
  - Manually building Docker image
  - Manually scanning for vulnerabilities
  - Manually pushing to ECR
  - Manually updating Kubernetes

Jenkins does all of this for us.

### Role of Jenkins in this DevSecOps Kubernetes Project
In the diagram, Jenkins sits between:
  - Source code (GitHub)
  - Docker
  - ECR
  - Security tools
  - ArgoCD

### Here is exactly what Jenkins does:

1. Trigger Pipeline on Code Push

When you push code to GitHub:

- Jenkins detects the chang
- Pipeline automatically starts

No human intervention needed.

2. Run Code Quality Analysis

Jenkins runs tools like: 
  - SonarQube (code quality check)
  - Linting tools

Purpose:
  - Check bad coding practices
  - Detect bugs early

3. Dependency & Security Check

Jenkins runs:
  - Dependency vulnerability scans
  - File system scans
  - Container scans (like Trivy)

This is the DevSecOps part. Security is checked before deployment.

4. Build Docker Image

Jenkins:
  - Builds Docker image
  - Tags the image (ex: v1.0.3)

5. Push Image to ECR

Jenkins:
  - Authenticates to AWS
  - Pushes image to ECR private repository

Now the image is stored securely in AWS.

6. Update Kubernetes Deployment File

Jenkins updates:
  - Kubernetes manifest
  - Image version tag

Then pushes changes back to GitHub.

7. Trigger ArgoCD Deployment
  - Argo CD watches GitHub.

When Jenkins updates the deployment file:
  - ArgoCD detects change
  - Automatically deploys new version to Kubernetes

## Step 1: We will create our Jenkins Server(EC2) on AWS.
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
      - Allow Custom TCP (9000) port Sonarqube
7. Storage
  - Make it 30G

8. Launch Instance 
  - Click Launch instance
  - Your EC2 will be running in ~30–60 seconds.

<img width="1555" height="257" alt="Screenshot 2026-02-07 at 12 23 28 PM" src="https://github.com/user-attachments/assets/c550c483-b895-4c04-92fb-b1dec90fe37a" />

### Why Do We Attach an IAM Role to Jenkins EC2?

When Jenkins runs on an EC2 server, it needs to:
  - Push Docker images to ECR
  - Create or update EKS resources
  - Access S3 (if used)
  - Describe clusters
  - Authenticate with AWS

- But AWS does not allow any machine to access services without permission.
- So we attach an IAM Role to the EC2 instance.

### What Is an IAM Role?
  - An IAM role is like a temporary permission badge for AWS services.
  - Instead of storing AWS access keys inside Jenkins (which is insecure), we attach a role to EC2.
  - Then Jenkins automatically gets permissions securely.

### Why AdministratorAccess?
During project setup (especially lab or learning project), people attach: AdministratorAccess

Because:
  - It gives full access to all AWS services
  - It avoids permission errors
  - It makes setup easier
  - Faster for demo / bootcamp projects

### What Jenkins Actually Needs Access To
In our DevSecOps Kubernetes project, Jenkins needs permission to:
  - ECR → push images
  - EKS → describe cluster
  - IAM → maybe assume roles
  - EC2 → describe instances
  - STS → get authentication tokens

- Is AdministratorAccess Recommended in Real Production? No.
- In real production: You should follow the Least Privilege Principle.

- That means: Only give permissions Jenkins truly needs. Example:
  - AmazonEC2ContainerRegistryFullAccess
  - AmazonEKSClusterPolicy
  - Custom IAM policy with limited actions

### Why Not Use Access Keys Instead?

Bad practice:
  - Storing AWS keys in Jenkins
  - Risk of leaks
  - Hard to rotate
  - Security issue

Best practice:
  - Attach IAM role to EC2
  - Jenkins automatically uses temporary credential
  - This is more secure.

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

### Why Do We Install Docker on the Jenkins Server?
Because Jenkins needs Docker to:
  - Build container images
  - Tag images
  - Push images to ECR

Without Docker installed, Jenkins cannot create container images.

### What Is Docker’s Role in This Project?

- Docker is used to package your application into a container image.
- Think of Docker like this: It puts your app + dependencies + runtime into one portable box.
- That box is called a Docker Image.

### Where Docker Fits in Your DevSecOps Flow
- Your project flow looks like this:
- Code → Jenkins → Docker → ECR → ArgoCD → Kubernetes

### Here’s what happens step-by-step:

1. Developer Pushes Code
  - Code goes to GitHub.
  - Jenkins pipeline starts.

2. Jenkins Runs Security & Quality Checks
  - Code analysis
  - Dependency scanning
  - File scanning
  - After checks pass…

3. Jenkins Uses Docker to Build Image
  - Jenkins runs:
```bash
docker build -t app:v1 .
```
- Docker:
  - Reads Dockerfile
  - Builds image
  - Packages app inside container
  - Now we have a deployable artifact.

4. Jenkins Pushes Image to ECR
```bash
docker push <ecr-repo>
```
The image is stored in AWS ECR.

5. Kubernetes Pulls Image
  - Kubernetes (via ArgoCD) pulls the image from ECR and runs it in pods.

### Why Not Deploy Code Directly?

Because:
  - Different environments behave differently
  - “It works on my machine” problem
  - Hard to scale

Docker solves this:
  - Same environment everywhere
  - Portable
  - Easy to scale in Kubernetes
  - Faster deployments

### What Happens If Docker Is NOT Installed on Jenkins?
  - Pipeline fails
  - Image cannot be built
  - Nothing to push to ECR
  - Kubernetes cannot deploy new version
  - Project stops at CI stage.

### Simple Summary
Docker’s role in your project:
  - Package the application
  - Create immutable image
  - Ensure consistency
  - Enable Kubernetes deployment

Jenkins needs Docker because Jenkins is responsible for building that image.

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

### Why Do We Need SonarQube?
- SonarQube is used to check the quality and security of source code before deployment.
- In DevSecOps, we don’t just deploy code.
- We make sure the code is:
  - Clean
  - Secure
  - Maintainable
  - Free of major bugs
  - That’s where SonarQube comes in.

### What Is the Role of SonarQube in Our Project?
- In your End-to-End DevSecOps Kubernetes pipeline: Code → Jenkins → SonarQube → Docker → ECR → ArgoCD → Kubernetes
- SonarQube runs during the CI stage

### What SonarQube Checks
- When Jenkins triggers the pipeline:
- SonarQube scans the code for:

1. Bugs
  - Logic mistakes that may break the application.

2. Vulnerabilities
  - Security issues like:
  - SQL injection risk
  - Hardcoded passwords
  - Unsafe code patterns

3. Code Smells
  - Bad coding practices (not clean or maintainable).

4. Code Coverage
- How much of your code is tested.

### Why Is This Important in DevSecOps?
- DevSecOps means: Security + DevOps together.
- Instead of finding security problems in production…
  - We detect them early in CI
  - We fail the build if quality is bad
  - We prevent insecure deployments

This saves money and prevents production incidents.

### What Happens If We Don’t Use SonarQube?
- Poor quality code goes to production
- Security vulnerabilities may be deployed
- Technical debt increases
- Harder to maintain later

### Why Are We Running SonarQube as a Docker Container on Jenkins Server?
- There are 3 main reasons:

1. Easy Installation
- Instead of:
  - Downloading SonarQube manually
  - Installing Java
  - Configuring services

We just run:
```bash
docker run -d -p 9000:9000 sonarqube
```

2. Isolation
- Running SonarQube in Docker:
  - Keeps it isolated from Jenkins
  - Avoids dependency conflicts
  - Easy to remove or restart

3. Lightweight for Learning Projects
- In small DevSecOps projects:
   - Jenkins runs on EC2
   - SonarQube runs as container on same EC2
This is cost-effective and simple.

### How Jenkins Connects to SonarQube
- Jenkins sends source code to SonarQube
- SonarQube analyzes code
- It returns a Quality Gate result
- If Quality Gate fails → Jenkins pipeline stops
- If passes → Continue to Docker build

## What Is Quality Gate?
- Quality Gate = pass or fail rule.
- Example:
  - No critical vulnerabilities
  - Code coverage above 70%
  - No blocker bugs
  - If rules fail → build fails.

### Simple Summary
- SonarQube’s role:
  - Analyze code quality
  - Detect security issues
  - Enforce quality gate
  - Prevent bad code from reaching Kubernetes

- We run it in Docker on Jenkins server because:
  - Easy setup
  - Isolated
  - Fast deployment
  - Good for small project environments

## Run Docker Container of Sonarqube
```bash
docker run -d  --name sonar -p 9000:9000 sonarqube:lts-community
```

### Why Do We Install AWS CLI on the Jenkins Server?
- Because Jenkins needs a way to talk to AWS.
- The tool that allows this communication is:
    - AWS Command Line Interface
    - AWS CLI lets Jenkins run AWS commands directly from the terminal or pipeline script.

### What Is the Role of AWS CLI in This DevSecOps Kubernetes Project?
- In our project: Code → Jenkins → Docker → ECR → EKS → ArgoCD → Kubernetes
- Jenkins needs to interact with AWS services like:
  - ECR (Elastic Container Registry)
  - EKS (Elastic Kubernetes Service)
  - IAM
  - STS
- AWS CLI makes this possible.

### What Jenkins Uses AWS CLI For
- Here’s exactly what happens:

1. Authenticate to ECR
- Before pushing Docker image, Jenkins runs:
```bash
aws ecr get-login-password
```
- This allows Docker to log in to ECR securely.
- Without AWS CLI → Docker cannot authenticate → push fails.

2. Push Docker Image to ECR
- After login, Jenkins pushes image.
- AWS CLI helps retrieve repository info and credentials.

3. Update kubeconfig for EKS
- To allow kubectl to talk to EKS cluster, Jenkins runs:
```css
aws eks update-kubeconfig --region us-east-1 --name my-cluster
```
- This connects Jenkins to your Kubernetes cluster.
- Without this → kubectl cannot access EKS.

4. Retrieve AWS Information
- Jenkins may use AWS CLI to:
  - Describe cluster
  - Get account ID
  - Get region info
  - Verify credentials
Example:
```sql
aws sts get-caller-identity
```

### Why Not Do Everything Manually?
- Because this is CI/CD automation.
- We want:
  - Automatic deployments
  - No manual AWS console clicks
  - Fully automated pipeline
- AWS CLI enables automation.

### What Happens If AWS CLI Is NOT Installed?
- Cannot log in to ECR
- Cannot update kubeconfig
- Cannot communicate with EKS
- Pipeline fails
- Your CI/CD breaks at AWS interaction stage.

### Important: How AWS CLI Gets Permissions
- Since you attached an IAM Role to Jenkins EC2:
- AWS CLI automatically uses temporary credentials from the EC2 instance role.
- No need to store access keys.
- This is secure.

### Simple Summary
- AWS CLI’s role in your project:
  - Authenticate to AWS services
  - Push Docker images to ECR
  - Connect to EKS cluster
  - Automate AWS operations
- Jenkins installs AWS CLI so it can control AWS programmatically inside the pipeline.is installed

### Install AWS CLI 
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

## What Is eksctl?
- eksctl is a command-line tool used to create and manage Amazon EKS clusters easily.
- Think of it like: A shortcut tool to create and manage EKS without writing everything manually.
- It works specifically with: Amazon EKS

### Why Do We Install eksctl on the Jenkins Server?
- Because Jenkins needs a way to:
  - Create EKS cluster
  - Create node groups
  - Manage IAM roles for EKS
  - Delete cluster (if needed)
  - Automate cluster provisioning
- Instead of manually creating EKS from AWS Console, Jenkins can run:
```bash
eksctl create cluster ...
```
- Fully automated

### What Is the Role of eksctl in Your DevSecOps Project?
- There are two possible scenarios:

Scenario 1: You Use eksctl to Create the Cluster
- In many bootcamp projects:
- Jenkins uses eksctl to:
  - Create EKS cluster
  - Create worker nodes
  - Configure IAM roles
  - Attach OIDC provider
- This makes cluster setup simple and fast.

Scenario 2: You Use Terraform for Infrastructure
- If your project already creates EKS using Terraform:
- Then eksctl might only be used for:
  - Managing node groups
  - Testing cluster creation manually
  - Quick cluster setup in labs
- In real production, usually:
  - Terraform OR CloudFormation handles infrastructure
  - eksctl is used more for quick setups

### Why Not Create EKS Manually in Console?
- Because this is DevOps.
- We want:
  - Automation
  - Repeatability
  - No manual clicks
  - Infrastructure as Code
- eksctl allows automation from Jenkins.

### What Happens If eksctl Is NOT Installed?
- If your pipeline uses eksctl commands:
  - Cluster cannot be created
  - Node groups cannot be added
  - Automation fails
- But…
  - If you are using Terraform for cluster creation
  - you technically don’t need eksctl in Jenkins.

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
### What Is kubectl?
- kubectl is the command-line tool used to communicate with a Kubernetes cluster.
- Think of it like: A remote control for Kubernetes.
- It works with: Kubernetes

### Why Do We Install kubectl on the Jenkins Server?
- Because Jenkins needs a way to:
  - Deploy applications to EKS
  - Apply Kubernetes manifests
  - Check pod status
  - Verify deployments
  - Rollout new versions
- Without kubectl, Jenkins cannot talk to the Kubernetes cluster.

### Role of kubectl in Our DevSecOps Project
- Your project flow: Code → Jenkins → Docker → ECR → EKS → Kubernetes
- kubectl is used after the Docker image is pushed to ECR.

### What Jenkins Uses kubectl For
1. Connect to EKS
- First Jenkins runs: aws eks update-kubeconfig ..
- This configures access to the cluster.
- Now kubectl can communicate with EKS.

2. Deploy Application
- Jenkins can run: kubectl apply -f deployment.yaml
- This deploys your application into the cluster.

3. Verify Deployment
- Jenkins can check: kubectl get pods, kubectl get svc, kubectl rollout status deployment/app
- This confirms that the deployment succeeded.

4. Troubleshooting (Optional)
- If deployment fails, Jenkins can: Get logs, Describe pods, Check events.

### If We Are Using ArgoCD, Do We Still Need kubectl?
- If your project uses: Argo CD
- Then:
  - Jenkins builds image
  - Updates Git repo
  - ArgoCD deploys automatically
- In that case:
  - kubectl may only be used for:
    - Initial cluster setup
    - Installing ArgoCD
    - Verification
    - Manual debugging
- In pure GitOps, Jenkins does NOT deploy directly using kubectl.
- ArgoCD handles deployment.

### What Happens If kubectl Is NOT Installed?
- If your pipeline uses kubectl commands:
  - Jenkins cannot deploy
  - Cannot check pods
  - Cannot validate cluster status
  - Deployment automation fails

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

### What Is Terraform?
- Terraform is a tool used to create and manage infrastructure using code.
- Instead of clicking in AWS Console…
- We write code like:
  - VPC
  - Subnets
  - EKS cluster
  - Security groups
  - IAM roles
- And Terraform creates them automatically.

### Why Do We Install Terraform on the Jenkins Server?
- Because Jenkins is our automation engine.
- If we want infrastructure to be:
  - Automated
  - Repeatable
  - Version-controlled
  - Created from pipeline
- Then Jenkins must run Terraform commands.
- Without Terraform installed, Jenkins cannot create AWS infrastructure.

### Role of Terraform in Our DevSecOps Kubernetes Project
- In our project:
  - Infrastructure Stage → Application Stage
  - Terraform handles the infrastructure stage.

### What Terraform Creates
- In our EKS DevSecOps project, Terraform typically creates:
  - VPC
  - Public & private subnets
  - Internet gateway
  - Route tables
  - Security groups
  - IAM roles
  - EKS cluster
  - Node groups
- All written in .tf files.

### How Jenkins Uses Terraform
- Inside the Jenkins pipeline: terraform init, terraform plan, terraform apply -auto-approve
- After infrastructure is ready:
  - Jenkins builds Docker image
  - Pushes to ECR
  - ArgoCD deploys to EKS

### Why Not Create Infrastructure Manually?
- Because manual setup is:
  - Slow
  - Error-prone
  - Not repeatable
  - Not version-controlled

### What Happens If Terraform Is NOT Installed?
- If your pipeline includes Terraform stage:
  - Jenkins cannot create EKS cluster
  - Cannot create VPC
  - Infrastructure provisioning fails
  - Whole pipeline breaks

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

### We will deploy our Three-Tier Application using ArgoCD.
As our repository is private. So, we need to configure the Private Repository in ArgoCD.

Step 1: Click on Settings and select Repositories

<img width="720" height="283" alt="image" src="https://github.com/user-attachments/assets/e757c6c6-c6fe-4762-a106-3d8044acd7b2" />

Step 2: Click on CONNECT REPO USING HTTPS

<img width="1600" height="628" alt="image" src="https://github.com/user-attachments/assets/87aafbad-2f7a-49b4-8d24-eeb96f3b44d9" />

Step 3: Provide the repository name where your Manifest files are present.
- Provide the username and GitHub Personal Access token and click on CONNECT.

<img width="1600" height="628" alt="image" src="https://github.com/user-attachments/assets/ee06ccef-08e1-43e3-98e6-7ec6866cfc42" />

Step 4: If your Connection Status is Successful, it means the repository connected successfully.

<img width="1600" height="207" alt="image" src="https://github.com/user-attachments/assets/de91ba8d-c55f-400c-bc61-44e4644fdfa2" />

### we will create our first application, which will be a database.

Step 1: We will create namaspace called three-tier in the eks cluste manually through jump server:
```bash
kubectl create namespace three-tier
```

Step 2: Click on CREATE APPLICATION.

<img width="720" height="209" alt="image" src="https://github.com/user-attachments/assets/7c2d2c80-e4d2-41b6-97bb-da2fb7667336" />

Step 3: Provide the details as it is provided in the snippet below and scroll down.

<img width="720" height="209" alt="image" src="https://github.com/user-attachments/assets/634d4cdd-b144-4c56-ab43-bf6fd8f7e4fa" />

- Select the same repository that you configured in the earlier step.
- In the Path, provide the location where your Manifest files are presented and provide other things as shown in the screenshot below.

Step 4: Click on CREATE.

<img width="1600" height="605" alt="image" src="https://github.com/user-attachments/assets/c26c1b02-5ccb-439e-b401-90e107dace50" />

### While your database Application is starting to deploy, we will create an application for the backend.

Step 1: Provide the details as it is provided in the snippet below and scroll down.

<img width="720" height="191" alt="image" src="https://github.com/user-attachments/assets/24b6f351-8edf-443e-95e8-666b8a75c602" />

- Select the same repository that you configured in the earlier step.
- In the Path, provide the location where your Manifest files are presented and provide other things as shown in the screenshot below.

Step 2: Click on CREATE.

<img width="720" height="282" alt="image" src="https://github.com/user-attachments/assets/75c35724-181d-4ab4-9a17-6bf2e1bf91de" />

### While your backend Application is starting to deploy, we will create an application for the frontend.

Step 1: Provide the details as it is provided in the snippet below and scroll down.

<img width="720" height="161" alt="image" src="https://github.com/user-attachments/assets/b5ed2957-c6fd-4683-bf4d-aa93b7e3870d" />

- Select the same repository that you configured in the earlier step.
- In the Path, provide the location where your Manifest files are presented and provide other things as shown in the screenshot below.

Step 2: Click on CREATE.

<img width="720" height="272" alt="image" src="https://github.com/user-attachments/assets/53ff4114-2a40-4bd4-96a6-91222cedd745" />

### While your frontend Application is starting to deploy, we will create an application for the ingress.

Step 1: Provide the details as it is provided in the snippet below and scroll down.

<img width="1600" height="353" alt="image" src="https://github.com/user-attachments/assets/86d1eb3a-12c8-4b43-b25c-d5f210db6a19" />

- Select the same repository that you configured in the earlier step.
- In the Path, provide the location where your Manifest files are presented and provide other things as shown in the screenshot below.

Step 2: Click on CREATE.

<img width="720" height="278" alt="image" src="https://github.com/user-attachments/assets/bc50e855-d76c-4658-b7fd-d9d70511cf8b" />

- Once your Ingress application is deployed. It will create an Application Load Balancer
- You can check out the load balancer named k8s-three.

<img width="720" height="150" alt="image" src="https://github.com/user-attachments/assets/d46d76c2-7ee6-4a11-9185-d8745ff2ee60" />

- Now, copy the ALB-DNS and go to your Domain Provider. In my case, Porkbun is the domain provider.
- Go to DNS and add a CNAME type with hostname backend, then add your ALB in the Answer and click on Save
- Note: I have created a subdomain backend.amanpathakdevops.study

<img width="1600" height="564" alt="image" src="https://github.com/user-attachments/assets/79d2d905-c362-464e-a968-73e2de341277" />

- You can see all 4 application deployments in the snippet below.

<img width="720" height="365" alt="image" src="https://github.com/user-attachments/assets/885a7d28-5870-458b-96ae-853c6e94f049" />

- Hit your subdomain after 2 to 3 minutes in your browser to see the magic.

<img width="720" height="379" alt="image" src="https://github.com/user-attachments/assets/938a15a1-a0a9-4e0d-acd5-5230e4ed4b59" />

- You can play with the application by adding the records.

<img width="720" height="379" alt="image" src="https://github.com/user-attachments/assets/29106efe-926b-4391-b6f6-48def0b3f04e" />

- You can play with the application by deleting the records.

### We will set up the Monitoring for our EKS Cluster. We can monitor the Cluster Specifications and other necessary things.

Step 1: We will achieve the monitoring using Helm. Add the Prometheus repo by using the command below
```bash
helm repo add stable https://charts.helm.sh/stable
```

<img width="1600" height="212" alt="image" src="https://github.com/user-attachments/assets/7c9107bd-de09-4bff-abc7-a9d56297698f" />

Step 2: Install the Prometheus
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/prometheus
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm install grafana grafana/grafana
```

<img width="720" height="107" alt="image" src="https://github.com/user-attachments/assets/05c0c6ee-b774-45b8-a6da-8376e3fa570f" />

Step 3: Now, check the service by the command below
```bash
kubectl get svc
kubectl get deploy
```

<img width="1600" height="237" alt="image" src="https://github.com/user-attachments/assets/8cce7652-04f1-46d9-87b2-402bcb61c4c7" />

### We will access our Prometheus and Grafana consoles from outside of the cluster.

Step 1: We need to change the Service type from ClusterType to LoadBalancer. Edit the stable-kube-prometheus-sta-prometheus service
```bash
kubectl edit svc stable-kube-prometheus-sta-prometheus
```
Modification in the 48th line from ClusterType to LoadBalancer

<img width="720" height="107" alt="image" src="https://github.com/user-attachments/assets/e68335ba-76d2-42d3-b70c-8780854dc596" />

Step 2: Edit the stable-grafana service
```bash
kubectl edit svc stable-grafana
```
Modification in the 39th line from ClusterType to LoadBalancer

<img width="720" height="80" alt="image" src="https://github.com/user-attachments/assets/682ae3cc-89fe-40ce-acd4-70d7e0471125" />

Step 3: Now, if you list the service again, then you will see the LoadBalancers' DNS names
```bash
kubectl get svc
```
<img width="720" height="89" alt="image" src="https://github.com/user-attachments/assets/3f0dab15-5857-4421-8e0c-d037eded0a73" />

Step 4: You can also validate from your console.

<img width="1600" height="293" alt="image" src="https://github.com/user-attachments/assets/643949c8-635d-4b77-9d45-eb7eb98fed6b" />

Step 5: Access your Prometheus Dashboard
```bash
Paste the <Prometheus-LB-DNS>:9090 in your favourite browser, and you will see it like this
```

<img width="720" height="172" alt="image" src="https://github.com/user-attachments/assets/5df887a0-ff1f-4be6-9791-db10597e4c82" />

Step 6: Click on Status and select Target. You will see a lot of Targets

<img width="720" height="378" alt="image" src="https://github.com/user-attachments/assets/3a6d4384-da13-4dab-b95e-c98abb13c2ca" />

### We will access your Grafana Dashboard

Step 1: Copy the ALB DNS of Grafana and paste it into your favourite browser. The username will be admin, and the password will be prom-operator for your Grafana LogIn.

<img width="720" height="378" alt="image" src="https://github.com/user-attachments/assets/f8d2bcec-604a-40af-b1d7-d102c36faa4f" />

Step 2: Now, click on Data Source

<img width="720" height="218" alt="image" src="https://github.com/user-attachments/assets/148b1ba0-4bc3-42ec-a45a-e56cc8ce3de4" />

Step 3: Select Prometheus

<img width="720" height="187" alt="image" src="https://github.com/user-attachments/assets/e6ed72b1-755c-4dad-9a3c-a73a7436fdfc" />

Step 4: In the Connection, paste your <Prometheus-LB-DNS>:9090.

<img width="720" height="279" alt="image" src="https://github.com/user-attachments/assets/5c78fc13-e51f-4515-a8a1-9415268947b5" />

Step 5: If the URL is correct, then you will see a green notification. Click on Save & test.

<img width="720" height="364" alt="image" src="https://github.com/user-attachments/assets/0bbbc0c3-0cd5-4f65-abb6-6a44d36f3fd9" />

Step 6: Now, we will create a dashboard to visualise our Kubernetes Cluster Logs. Click on Dashboard.

<img width="720" height="87" alt="image" src="https://github.com/user-attachments/assets/e4aad5f0-6e84-4f03-98cb-fb06b2aa86e4" />

Step 7: Once you click on Dashboard. You will see a lot of Kubernetes components being monitored.

<img width="720" height="366" alt="image" src="https://github.com/user-attachments/assets/7acc006a-b795-47ec-8c3b-7c6e0b4d6d28" />

Step 8: Let’s try to import a type of Kubernetes Dashboard. Click on New and select Import

<img width="720" height="111" alt="image" src="https://github.com/user-attachments/assets/38fd4a51-e5e8-4f3e-913f-242f265ec3c7" />

Step 9: Provide 6417 ID and click on Load
- Note: 6417 is a unique ID from Grafana, which is used to monitor and visualise Kubernetes Data

<img width="720" height="286" alt="image" src="https://github.com/user-attachments/assets/2380bf9a-4c1e-45e7-95f6-8f96720e5e02" />

Step 10: Select the data source that you have created earlier and click on Import.

<img width="720" height="294" alt="image" src="https://github.com/user-attachments/assets/fa52636c-953f-48bf-98c6-306f8b6fe915" />

Step 11: Here, you go. You can view your Kubernetes Cluster Data. Feel free to explore the other details of the Kubernetes Cluster.

<img width="720" height="366" alt="image" src="https://github.com/user-attachments/assets/f8f2b85b-940f-42e6-868c-ea189763e70a" />


### Conclusion
- In this comprehensive DevSecOps Kubernetes project, we successfully:
  - Established IAM user and Terraform for AWS setup.
  - Deployed Jenkins on AWS, configured tools, and integrated it with SonarQube.
  - Set up an EKS cluster, configured a Load Balancer, and established private ECR repositories.
  - Implemented monitoring with Helm, Prometheus, and Grafana.
  - Installed and configured ArgoCD for GitOps practices.
  - Created Jenkins pipelines for CI/CD, deploying a Three-Tier application.
  - Ensured data persistence with persistent volumes and claims.


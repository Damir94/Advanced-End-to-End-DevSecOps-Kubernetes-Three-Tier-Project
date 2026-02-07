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





### Configure Terraform
- Edit the file /etc/environment using the below command, add the highlighted lines and add your keys in the blur space.
```bash
sudo vim /etc/environment
```
- After making the changes, restart your machine to reflect the changes to your environment variables.

### Configure AWS CLI
- Run the below command, and add your keys
```bash
aws configure
```

### Step 3: Deploy the Jenkins Server(EC2) using Terraform
- Clone the Git repository
```bash
https://github.com/AmanPathak-DevOps/End-to-End-Kubernetes-Three-Tier-DevSecOps-Project
```
- Navigate to the Jenkins-Server-TF

- Do some modifications to the backend.tf file, such as changing the bucket name and dynamodb table(make sure you have created both manually on AWS Cloud).

- Now, you have to replace the PEM file name as you have some other name for your PEM file. To provide the PEM file name that is already created on AWS

- Initialise the backend by running the command below
```bash
terraform init
```
- Run the command below to check the syntax error
```bash
terraform validate
```
- Run the below command to get the blueprint of what kind of AWS services will be created.
```bash
terraform plan -var-file=variables.tfvars
```
- Now, run the below command to create the infrastructure on AWS Cloud, which will take 3 to 4 minutes maximum
```bash
terraform apply -var-file=variables.tfvars --auto-approve
```
- Now, connect to your Jenkins server by clicking on Connect.

- Copy the SSH command and paste it on your local machine.

### Step 4: Configure the Jenkins
- Now, we logged into our Jenkins server.

- We have installed some services such as Jenkins, Docker, Sonarqube, Terraform, Kubectl, AWS CLI, and Trivy.
- Let’s validate whether all our installed or not.
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
- The Credentials will look like the snippet below.
- Now, we need to add GitHub credentials as well because currently, my repository is Private.
- This thing, I am performing this because in Industry Projects, your repository will be private.
- So, add the username and personal access token of your GitHub account.
- Both credentials will look like this.
- Create an eks cluster using the commands below.
```bash
eksctl create cluster --name Three-Tier-K8s-EKS-Cluster --region us-east-1 --node-type t2.medium --nodes-min 2 --nodes-max 2
aws eks update-kubeconfig --region us-east-1 --name Three-Tier-K8s-EKS-Cluster
```
Once your cluster is created, you can validate whether your nodes are ready or not by using the following command
```bash
kubectl get nodes
```
### Step 6: Now, we will configure the Load Balancer on our EKS because our application will have an ingress controller.
- Download the policy for the LoadBalancer prerequisite.
```bash
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
```
- Create the IAM policy using the command below
```bash
aws iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam_policy.json
```
- Create OIDC Provider
```bash
eksctl utils associate-iam-oidc-provider --region=us-east-1 --cluster=Three-Tier-K8s-EKS-Cluster --approve
```
- Create a Service Account by using the below command and replace your account ID with your one
```bash
eksctl create iamserviceaccount --cluster=Three-Tier-K8s-EKS-Cluster --namespace=kube-system --name=aws-load-balancer-controller --role-name AmazonEKSLoadBalancerControllerRole --attach-policy-arn=arn:aws:iam::<your_account_id>:policy/AWSLoadBalancerControllerIAMPolicy --approve --region=us-east-1
```
- Run the below command to deploy the AWS Load Balancer Controller
```bash
sudo snap install helm --classic
helm repo add eks https://aws.github.io/eks-charts
helm repo update eks
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=my-cluster --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer-controller
```
- After 2 minutes, run the command below to check whether your pods are running or not.
```bash
kubectl get deployment -n kube-system aws-load-balancer-controller
```
- If the pods are getting Error or CrashLoopBackOff, then use the below command
```bash
helm upgrade -i aws-load-balancer-controller eks/aws-load-balancer-controller \
  --set clusterName=<cluster-name> \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=us-west-1 --set vpcId=<vpc#> -n kube-system
```
### Step 7: We need to create Amazon ECR Private Repositories for both Tiers (Frontend & Backend)
- Click on Create repository
- Select the Private option to provide the repository and click on Save.
- Do the same for the backend repository and click on Save
- Now, we have set up our ECR Private Repository and
- Now, we need to configure ECR locally because we have to upload our images to Amazon ECR.
- Copy the 1st command for login
- Now, run the copied command on your Jenkins Server.

### Step 8: Install & Configure ArgoCD
- We will be deploying our application on a three-tier namespace. To do that, we will create a three-tier namespace on EKS
```bash
kubectl create namespace three-tier
```
- As you know, our two ECR repositories are private. So, when we try to push images to the ECR Repos, it will give us the error ImagePullError.
- To get rid of this error, we will create a secret for our ECR Repo by the below command and then, we will add this secret to the deployment file.
- Note: The Secrets are coming from the .docker/config.json file, which is created while logging the ECR in the earlier steps
```bash
kubectl create secret generic ecr-registry-secret \
  --from-file=.dockerconfigjson=${HOME}/.docker/config.json \
  --type=kubernetes.io/dockerconfigjson --namespace three-tier
kubectl get secrets -n three-tier
```
- Now, we will install argoCD.
- To do that, create a separate namespace for it and apply the argocd configuration for installation.
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.4.7/manifests/install.yaml
```
- All pods must be running. To validate, run the command below
```bash
kubectl get pods -n argocd
```
- Now, expose the argoCD server as a LoadBalancer using the below command
```bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```
- You can validate whether the Load Balancer is created or not by going to the AWS Console
- To access the argoCD, copy the LoadBalancer DNS and hit it on your favourite browser.
- You will get a warning like the snippet below.
- Click on Advanced.
- Click on the link below, which is appearing under Hide advanced
- Now, we need to get the password for our argoCD server to perform the deployment.
- To do that, we have a prerequisite, which is jq. Install it by the command below.
```bash
sudo apt install jq -y
```
```bash
export ARGOCD_SERVER=$(kubectl get svc argocd-server -n argocd -o json | jq -r '.status.loadBalancer.ingress[0].hostname')
export ARGO_PWD=$(kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d)
```
- Enter the username and password in argoCD and click on SIGN IN.
- Here is our ArgoCD Dashboard.

### Step 9: Now, we have to configure SonarQube for our DevSecOps Pipeline
- To do that, copy your Jenkins Server public IP and paste it into your favourite browser with a 9000 port
- The username and password will be admin
- Click on Log In.
- Update the password
- Click on Administration, then Security, and select Users
- Click on Update tokens
- Click on Generate
- Copy the token, keep it somewhere safe and click on Done.
- Now, we have to configure webhooks for quality checks.
- Click on Administration, then Configuration, and select Webhooks
- Click on Create
- Provide the name of your project and in the URL, provide the Jenkins server public IP with port 8080, add sonarqube-webhook in the suffix, and click on Create.
```bash
http://<jenkins-server-public-ip>:8080/sonarqube-webhook/
```
- Here, you can see the webhook.
- Now, we have to create a Project for the frontend code.
- Click on Manually.
- Provide the display name to your Project and click on Setup
- Click on Locally.
- Select the Use existing token and click on Continue.
- Select Other and Linux as OS.
- After performing the above steps, you will get the command, which you can see in the snippet below.
- Now, use the command in the Jenkins Frontend Pipeline where Code Quality Analysis will be performed.
- Now, we have to create a Project for the backend code.
- Click on Create Project.
- Provide the name of your project and click on Set up.
- Click on Locally.
- Select the Use existing token and click on Continue.
- Select Other and Linux as OS.
- After performing the above steps, you will get the command, which you can see in the snippet below.
- Now, use the command in the Jenkins Frontend Pipeline where Code Quality Analysis will be performed.
- Now, we have to create a Project for the backend code.
- Click on Create Project.
- Provide the name of your project and click on Set up.
- Click on Locally.
- Select the Use existing token and click on Continue.
- Select Other and Linux as OS.
- After performing the above steps, you will get the command, which you can see in the snippet below.
- Now, use the command in the Jenkins Backend Pipeline where Code Quality Analysis will be performed.
- Now, we have to store the sonar credentials.
- Go to Dashboard -> Manage Jenkins -> Credentials
- Select the kind as Secret text, paste your token in Secret and keep other things as it is.
- Click on Create
- Now, we have to store the GitHub Personal access token to push the deployment file, which will be modified in the pipeline itself for the ECR image.
- Add GitHub credentials
- Select the kind as Secret text and paste your GitHub Personal access token(not password) in Secret, and keep other things as it is.
- Click on Create
- Note: If you haven’t generated your token, you generate it first, then paste it into the Jenkins
- Now, according to our Pipeline, we need to add an Account ID in the Jenkins credentials because of the ECR repo URI.
- Select the kind as Secret text, paste your AWS Account ID in Secret and keep other things as it is.
- Click on Create
- Now, we need to provide our ECR image name for the frontend, which is frontend only.
- Select the kind as Secret text, paste your frontend repo name in Secret and keep other things as it is.
- Click on Create
- Now, we need to provide our ECR image name for the backend, which is backend only.
- Select the kind as Secret text, paste your backend repo name in Secret, and keep other things as it is.
- Click on Create
- Final Snippet of all Credentials that we needed to implement this project.

### Step 10: Install the required plugins and configure the plugins to deploy our Three-Tier Application
- Install the following plugins by going to Dashboard -> Manage Jenkins -> Plugins -> Available Plugins
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
- Now, we have to configure the installed plugins.
- Go to Dashboard -> Manage Jenkins -> Tools
- We are configuring JDK
- Search for JDK and provide the configuration like the snippet below.
- Now, we will configure the SonarQube scanner
- Search for the SonarQube scanner and provide the configuration like the snippet below.
- Now, we will configure nodejs
- Search for the node and provide the configuration, like the snippet below.
- Now, we will configure the OWASP Dependency Check
- Search for Dependency-Check and provide the configuration like the snippet below.
- Now, we will configure the Docker
- Search for Docker and provide the configuration like the snippet below.
- Now, we have to set the path for SonarQube in Jenkins
- Go to Dashboard -> Manage Jenkins -> System
- Search for SonarQube installations
- Provide the name as it is, then in the Server URL, copy the SonarQube public IP (same as Jenkins) with port 9000, select the Sonar token that we have added recently, and click on Apply & Save.
- Now, we are ready to create our Jenkins Pipeline to deploy our Backend Code.
- Go to Jenkins Dashboard
- Click on New Item
- Provide the name of your Pipeline and click on OK.
- This is the Jenkins file to deploy the Backend Code on EKS.
- Copy and paste it into the Jenkins
```bash
https://github.com/AmanPathak-DevOps/End-to-End-Kubernetes-Three-Tier-DevSecOps-Project/blob/master/Jenkins-Pipeline-Code/Jenkinsfile-Backend
```
- Click Apply & Save.
- Now, click on the build.
- Our pipeline was successful after addressing a few common mistakes.
- Note: Do the changes in the Pipeline according to your project.
- Now, we are ready to create our Jenkins Pipeline to deploy our Frontend Code.
- Go to Jenkins Dashboard
- Click on New Item
- Provide the name of your Pipeline and click on OK.
- This is the Jenkins file to deploy the Frontend Code on EKS.
- Copy and paste it into the Jenkins
```bash
https://github.com/AmanPathak-DevOps/End-to-End-Kubernetes-Three-Tier-DevSecOps-Project/blob/master/Jenkins-Pipeline-Code/Jenkinsfile-Frontend
```
- Click Apply & Save.
- Now, click on the build.
- Our pipeline was successful after a few common mistakes.
- Note: Do the changes in the Pipeline according to your project.

### Setup 11: We will set up the Monitoring for our EKS Cluster. We can monitor the Cluster Specifications and other necessary things.
- We will achieve the monitoring using Helm
- Add the Prometheus repo by using the command below
```bash
helm repo add stable https://charts.helm.sh/stable
```
- Install the Prometheus
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/prometheus
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm install grafana grafana/grafana
```
- Now, check the service by the command below
```bash
kubectl get svc
```
- Now, we need to access our Prometheus and Grafana consoles from outside of the cluster.
- For that, we need to change the Service type from ClusterType to LoadBalancer
- Edit the stable-kube-prometheus-sta-prometheus service
```bash
kubectl edit svc stable-kube-prometheus-sta-prometheus
```
- Modification in the 48th line from ClusterType to LoadBalancer
- Edit the stable-grafana service
```bash
kubectl edit svc stable-grafana
```
- Modification in the 39th line from ClusterType to LoadBalancer
- Now, if you list the service again, then you will see the LoadBalancers' DNS names
```bash
kubectl get svc
```
- You can also validate from your console.
- Now, access your Prometheus Dashboard
- Paste the <Prometheus-LB-DNS>:9090 in your favourite browser, and you will see it like this
- Click on Status and select Target.
- You will see a lot of Targets
- Now, access your Grafana Dashboard
- Copy the ALB DNS of Grafana and paste it into your favourite browser.
- The username will be admin, and the password will be prom-operator for your Grafana LogIn.
- Now, click on Data Source
- Select Prometheus
- In the Connection, paste your <Prometheus-LB-DNS>:9090.
- If the URL is correct, then you will see a green notification/
- Click on Save & test.
- Now, we will create a dashboard to visualise our Kubernetes Cluster Logs.
- Click on Dashboard.
- Once you click on Dashboard. You will see a lot of Kubernetes components being monitored.
- Let’s try to import a type of Kubernetes Dashboard.
- Click on New and select Import
- Provide 6417 ID and click on Load
- Note: 6417 is a unique ID from Grafana, which is used to monitor and visualise Kubernetes Data
- Select the data source that you have created earlier and click on Import.
- Here, you go.
- You can view your Kubernetes Cluster Data.
- Feel free to explore the other details of the Kubernetes Cluster.

### Step 12: We will deploy our Three-Tier Application using ArgoCD.
- As our repository is private. So, we need to configure the Private Repository in ArgoCD.
- Click on Settings and select Repositories
- Click on CONNECT REPO USING HTTPS
- Now, provide the repository name where your Manifest files are present.
- Provide the username and GitHub Personal Access token and click on CONNECT.
- If your Connection Status is Successful, it means the repository connected successfully.
- Now, we will create our first application, which will be a database.
- Click on CREATE APPLICATION.
- Provide the details as it is provided in the snippet below and scroll down.
- Select the same repository that you configured in the earlier step.
- In the Path, provide the location where your Manifest files are presented and provide other things as shown in the screenshot below.
- Click on CREATE.
- While your database Application is starting to deploy, we will create an application for the backend.
- Provide the details as it is provided in the snippet below and scroll down.
- Select the same repository that you configured in the earlier step.
- In the Path, provide the location where your Manifest files are presented and provide other things as shown in the screenshot below.
- Click on CREATE.
- While your backend Application is starting to deploy, we will create an application for the frontend.
- Provide the details as it is provided in the snippet below and scroll down.
- Select the same repository that you configured in the earlier step.
- In the Path, provide the location where your Manifest files are presented and provide other things as shown in the screenshot below.
- Click on CREATE.
- While your frontend Application is starting to deploy, we will create an application for the ingress.
- Provide the details as it is provided in the snippet below and scroll down.
- Select the same repository that you configured in the earlier step.
- In the Path, provide the location where your Manifest files are presented and provide other things as shown in the screenshot below.
- Click on CREATE.
- Once your Ingress application is deployed. It will create an Application Load Balancer
- You can check out the load balancer named k8s-three.
- Now, copy the ALB-DNS and go to your Domain Provider. In my case, Porkbun is the domain provider.
- Go to DNS and add a CNAME type with hostname backend, then add your ALB in the Answer and click on Save
- Note: I have created a subdomain backend.amanpathakdevops.study
- You can see all 4 application deployments in the snippet below.
- Now, hit your subdomain after 2 to 3 minutes in your browser to see the magic.
- You can play with the application by adding the records.
- You can play with the application by deleting the records.
- Now, you can see your Grafana Dashboard to view the EKS data, such as pods, namespace, deployments, etc.
- If you want to monitor the three-tier namespace.
- In the namespace, replace three-tier with another namespace.
- You will see the deployments that are done by ArgoCD
- This is the Ingress Application Deployment in ArgoCD
- This is the Frontend Application Deployment in ArgoCD
- This is the Backend Application Deployment in ArgoCD
- This is the Database Application Deployment in ArgoCD
- If you observe, we have configured the Persistent Volume & Persistent Volume Claim. So, if the pods get deleted, then the data won’t be lost. The Data will be stored on the host machine.
- To validate it, delete both Database pods.
- Now, the new pods will be started.
- And Your Application won’t lose a single piece of data.

### Conclusion
- In this comprehensive DevSecOps Kubernetes project, we successfully:
  - Established IAM user and Terraform for AWS setup.
  - Deployed Jenkins on AWS, configured tools, and integrated it with SonarQube.
  - Set up an EKS cluster, configured a Load Balancer, and established private ECR repositories.
  - Implemented monitoring with Helm, Prometheus, and Grafana.
  - Installed and configured ArgoCD for GitOps practices.
  - Created Jenkins pipelines for CI/CD, deploying a Three-Tier application.
  - Ensured data persistence with persistent volumes and claims.


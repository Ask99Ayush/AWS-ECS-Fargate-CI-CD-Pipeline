# Automated CI/CD Pipeline using Jenkins, Docker, AWS ECS (Fargate) & Application Load Balancer

## Complete Implementation Guide with Step-by-Step Instructions

---

## 📑 Table of Contents

1. [Introduction to the Project](#1-introduction-to-the-project)
   - 1.1 [What Is This Project?](#11-what-is-this-project)
   - 1.2 [Real-World Analogy](#12-real-world-analogy)
   - 1.3 [Why This Project Matters in Industry](#13-why-this-project-matters-in-industry)

2. [Why This Project Is Needed](#2-why-this-project-is-needed-problem-explanation)
   - 2.1 [Traditional Deployment (Manual Way)](#21-traditional-deployment-manual-way)
   - 2.2 [Problems with Manual Deployment](#22-problems-with-manual-deployment)
   - 2.3 [Why Automation Is Mandatory](#23-why-automation-is-mandatory)

3. [Understanding CI/CD](#3-understanding-cicd-beginner--advanced)
   - 3.1 [Continuous Integration (CI)](#31-continuous-integration-ci)
   - 3.2 [Continuous Deployment (CD)](#32-continuous-deployment-cd)
   - 3.3 [CI vs Continuous Delivery vs Continuous Deployment](#33-ci-vs-continuous-delivery-vs-continuous-deployment)
   - 3.4 [Why CI/CD Is an Industry Standard](#34-why-cicd-is-an-industry-standard)

4. [What This Project Solves](#4-what-this-project-solves)

5. [Benefits of This Project](#5-benefits-of-this-project)

6. [Tools & Technologies](#6-tools--technologies)
   - 6.1 [GitHub](#61-github)
   - 6.2 [Jenkins](#62-jenkins)
   - 6.3 [Docker](#63-docker)
   - 6.4 [AWS ECR](#64-aws-ecr)
   - 6.5 [AWS ECS (Fargate)](#65-aws-ecs-fargate)
   - 6.6 [Application Load Balancer](#66-application-load-balancer)

7. [High-Level Architecture](#7-high-level-architecture)

8. [Internal Data & Control Flow](#8-internal-data--control-flow)

9. [Deployment Strategies & Zero Downtime](#9-deployment-strategies--zero-downtime)

10. [Security Best Practices & Common Mistakes](#10-security-best-practices--common-mistakes)

11. [Implementation Guide (Step-by-Step)](#11-implementation-guide)
    - [STEP 1: Create EC2 Instance for Jenkins Server](#step-1-create-ec2-instance-for-jenkins-server)
    - [STEP 2: IAM Role Setup for Jenkins](#step-2-iam-role-setup-for-jenkins-ec2--ecr--ecs)
    - [STEP 3: Prepare EC2 Server for Jenkins & CI/CD](#step-3-prepare-ec2-server-for-jenkins--cicd)
    - [STEP 4: Install & Configure Jenkins on EC2](#step-4-install--configure-jenkins-on-ec2)
    - [STEP 5: Clone Application Repository in Jenkins](#step-5-clone-application-repository-in-jenkins)
    - [STEP 6: Build Docker Image using Jenkins Pipeline](#step-6-build-docker-image-using-jenkins-pipeline-core-ci-step)
    - [STEP 7: Push Docker Image to Amazon ECR](#step-7-push-docker-image-to-amazon-ecr)
    - [STEP 8: Create ECS Cluster (AWS Fargate)](#step-8-create-ecs-cluster-aws-fargate)
    - [STEP 9: Create ECS Task Definition (Fargate)](#step-9-create-ecs-task-definition-fargate)
    - [STEP 10: Create ECS Service (AWS Fargate)](#step-10-create-ecs-service-aws-fargate)
    - [STEP 11: Create Application Load Balancer (ALB)](#step-11-create-application-load-balancer-alb)
    - [STEP 12: Create Target Group for ECS Service](#step-12-create-target-group-for-ecs-service)
    - [STEP 13: Attach Target Group to ECS Service](#step-13-attach-target-group-to-ecs-service-enable-traffic-flow)


12. [Monitoring & Observability (Production Readiness)](#12-monitoring--observability-production-readiness)
13. [Final Takeaway](#13-final-takeaway)

---

## 1. Introduction to the Project

### 1.1 What Is This Project?
This project demonstrates how to build a real‑world, production‑grade CI/CD pipeline that automatically deploys a containerized application to AWS using modern DevOps tools.

**In simple terms**, whenever a developer pushes code to GitHub, the system automatically:
- Builds the application
- Packages it into a Docker container
- Stores it securely in AWS ECR
- Deploys it to AWS ECS (Fargate)
- Makes it available to users via a stable Application Load Balancer URL

All of this happens **without any manual intervention** using a CI/CD (Continuous Integration & Continuous Deployment) pipeline.

### 1.2 Real‑World Analogy
Think of this pipeline as a fully automated factory:
- **GitHub** → Raw material arrives (source code)
- **Jenkins** → Factory manager coordinating all tasks
- **Docker** → Packaging unit creating consistent containers
- **AWS ECR** → Secure warehouse for container images
- **AWS ECS (Fargate)** → Delivery vehicles running your application
- **Application Load Balancer** → Smart traffic controller directing users to healthy deliveries

Once configured, every new code version flows automatically to end users.

### 1.3 Why This Project Matters in Industry
This is **not a toy project**. The same architecture is used by:
- **SaaS companies** (delivering features continuously)
- **Startups** (scaling rapidly without infrastructure headaches)
- **Large enterprises** (modernizing legacy applications)
- **Microservices‑based platforms** (managing multiple services)

Understanding this project means understanding:
- How real deployments happen in production
- How DevOps teams reduce deployment risk
- How zero‑downtime releases are achieved
- How cloud-native applications are built and maintained

---

## 2. Why This Project Is Needed (Problem Explanation)

### 2.1 Traditional Deployment (Manual Way)
In traditional setups:
1. Developer updates code and commits to repository
2. Application is built manually on a server
3. Docker image is created manually using docker build commands
4. Image is pushed manually to a registry
5. ECS task definition is updated manually
6. ECS services are restarted manually
7. Health checks are verified manually

This approach **does not scale** beyond simple applications.

### 2.2 Problems with Manual Deployment
❌ **Time-consuming** – Every deployment takes 15-30 minutes of manual work  
❌ **Human errors** – Typos, wrong tags, forgotten steps cause failures  
❌ **No consistency** – Different team members deploy differently  
❌ **Not scalable** – Can't handle multiple services or frequent updates  
❌ **No rollback safety** – Reverting means repeating all manual steps  

**Manual deployment in production is risky and unprofessional.**

### 2.3 Why Automation Is Mandatory
Modern applications:
- Change frequently (multiple times per day)
- Serve many users (scalability required)
- Must stay online 24/7 (no maintenance windows)

**Automation ensures:**
- **Speed** – Deploy in minutes, not hours
- **Safety** – Same process every time
- **Repeatability** – Consistent across environments
- **Reliability** – No human mistakes

---

## 3. Understanding CI/CD (Beginner → Advanced)

### 3.1 Continuous Integration (CI)
**Definition:** Automatically building and validating code whenever changes are pushed to the repository.

**CI internally:**
1. Pulls latest code from GitHub
2. Builds the application (compile, package)
3. Runs validations (tests, linting)
4. Creates a deployable artifact (Docker image)
5. Ensures code is always deployable

**Why CI matters:**
- Detects integration errors early
- Keeps main branch stable
- Provides fast feedback to developers
- Automates repetitive build tasks

### 3.2 Continuous Deployment (CD)
**Definition:** Automatically deploying verified code to production **without manual approval**.

**Once CI succeeds:**
1. New version is deployed automatically
2. Zero-downtime deployment ensures no disruption
3. Users get updates immediately
4. Rollback is automated if issues detected

### 3.3 CI vs Continuous Delivery vs Continuous Deployment

| Term | Meaning | Approval Required? |
|------|---------|-------------------|
| **CI** | Build & validate code automatically | No |
| **Continuous Delivery** | Ready for deployment, waiting for manual approval | Yes (manual trigger) |
| **Continuous Deployment** | Auto‑deploy to production immediately | No |

**This project uses Continuous Deployment** – fully automated from commit to production.

### 3.4 Why CI/CD Is an Industry Standard
- **Faster releases** – From weeks to hours
- **Safer deployments** – Consistent, tested process
- **Easier rollbacks** – Versioned artifacts enable quick revert
- **Better collaboration** – Everyone follows same process

---

## 4. What This Project Solves

This project builds a fully automated deployment pipeline that:
- **Eliminates all manual steps** – Code push triggers everything
- **Uses containers for consistency** – Same environment everywhere
- **Runs on serverless infrastructure** – No servers to manage (Fargate)
- **Provides stable access** – Application Load Balancer gives fixed URL
- **Reflects real DevOps workflows** – Exactly what companies use
- **Teaches industry best practices** – IAM roles, no hardcoded credentials

---

## 5. Benefits of This Project

✅ **Fully automated deployments** – Push code, see it live  
✅ **Zero‑downtime updates** – Users never see "Under Maintenance"  
✅ **Scalable container architecture** – Handles traffic spikes automatically  
✅ **Stable DNS endpoint** – ALB URL never changes  
✅ **Secure image storage** – Private ECR with IAM security  
✅ **Serverless execution** – No EC2 management, AWS handles capacity  
✅ **Strong DevOps foundation** – Skills transferable to any company  
✅ **Interview-ready knowledge** – Explain every component with confidence  
✅ **Production-grade patterns** – Same architecture used by Fortune 500 companies  

---

## 6. Tools & Technologies

### 6.1 GitHub
**Role:** Source code repository and pipeline trigger
- Stores application source code and Dockerfile
- Sends webhook notifications when code changes
- Maintains version history for rollbacks
- **Why:** Industry-standard Git hosting

### 6.2 Jenkins
**Role:** CI/CD orchestrator (the brain)
- Executes pipeline stages sequentially
- Pulls code from GitHub
- Builds Docker images
- Interacts with AWS services
- **Why:** Most widely used automation server

### 6.3 Docker
**Role:** Containerization platform
- Solves "it works on my machine" problem
- Provides consistent runtime environment
- Packages application with dependencies
- Ideal for microservices architecture
- **Why:** Industry standard for containerization

### 6.4 AWS ECR (Elastic Container Registry)
**Role:** Private Docker image registry
- Securely stores Docker images
- IAM-based authentication (no passwords)
- Natively integrated with ECS
- **Why:** Secure, managed, no rate limits

### 6.5 AWS ECS (Fargate)
**Role:** Container orchestration (serverless)
- Runs containers without managing servers
- Auto-heals failed containers
- Scales based on demand
- **Why:** Serverless = less operational overhead

### 6.6 Application Load Balancer (ALB)
**Role:** Traffic distribution
- Provides stable public endpoint
- Distributes traffic across containers
- Performs health checks
- Enables zero-downtime deployments
- **Why:** Production-grade traffic management

## 7. High-Level Architecture
![Automated CI/CD Pipeline with Monitoring and Alerting using Jenkins, Docker, AWS ECS (Fargate) and ALB](./docs/architecture_diagram/architecture_diagram.png)


## 8. Internal Data & Control Flow

1. Developer pushes code → GitHub
2. GitHub webhook → Jenkins
3. Jenkins pulls code → Jenkins workspace
4. Jenkins builds Docker image → docker build
5. Jenkins pushes image → Amazon ECR
6. Jenkins triggers ECS deployment → update-service
7. ECS pulls new image → Fargate launches containers
8. ALB registers new containers → health checks
9. ALB shifts traffic → new containers receive traffic
10. Users access app → via ALB DNS

## 9. Deployment Strategies & Zero Downtime

### Rolling Deployment
- Gradual replacement of old containers with new ones
- Default ECS behavior
- Maintains availability throughout update

### Blue‑Green Deployment
- Two environments (blue = current, green = new)
- Switch traffic once green is healthy
- Safer but more complex infrastructure

### Zero‑Downtime Concept
Achieved using:
- **Parallel containers** – New version runs alongside old
- **ALB health checks** – Traffic only to healthy containers
- **Traffic shifting** – Gradually move users to new version
- **No single point of failure** – Multiple containers always running

---

## 10. Security Best Practices & Common Mistakes

### ✅ Best Practices
| Practice | Why |
|----------|-----|
| **IAM roles with least privilege** | Only necessary permissions |
| **No hardcoded credentials** | Never store AWS keys in code |
| **Private ECR repository** | Images not publicly accessible |
| **Jenkins credentials store** | Secure storage for secrets |
| **Separate security groups** | ALB and ECS tasks isolated |
| **Health checks** | Ensure only healthy containers receive traffic |

### ❌ Common Mistakes
⚠️ Using AWS access keys in Jenkins instead of IAM roles  
⚠️ Tagging images as `latest` only (no versioning)  
⚠️ Skipping health checks or misconfiguring them  
⚠️ Manually restarting ECS services (defeats automation)  
⚠️ Hardcoding values in Jenkinsfile  
⚠️ Using wrong target type (Instance vs IP) for ECS  
⚠️ Forgetting execution role in task definition  
⚠️ Not verifying IAM role attachment to EC2  

---

## 11. Implementation Guide

---

## STEP 1: Create EC2 Instance for Jenkins Server

### Step 1 Overview
**What Are We Doing?**
Creating an EC2 virtual machine that will act as the Jenkins server to run our CI/CD pipeline 24/7.

### 📋 Prerequisites
- AWS Account with appropriate permissions
- SSH key pair for EC2 access

---

### METHOD 1: GUI IMPLEMENTATION (AWS Console)

#### Step 1.1: Open EC2 Launch Wizard
**Path:** AWS Management Console → EC2 → Instances → Launch instance

#### Step 1.2: Configure Basic Instance Details

| Setting | Value | Why |
|---------|-------|-----|
| **Name** | `jenkins-server` | Easy identification |
| **AMI** | Ubuntu Server 22.04 LTS (64-bit x86) | Stable, Docker-friendly |
| **Instance type** | `t2.micro` or `t3.micro` | Free tier eligible |

#### Step 1.3: Key Pair Configuration
- **Option 1:** Select existing key pair
- **Option 2:** Create new key pair
  - Name: `jenkins-key`
  - Type: RSA
  - Format: `.pem`
  - **⚠️ Download and store safely – cannot be recovered**

#### Step 1.4: Network Settings (Security Group)
Create new security group with these inbound rules:

| Type | Protocol | Port | Source | Purpose |
|------|----------|------|--------|---------|
| SSH | TCP | 22 | Your IP | Secure login |
| Custom TCP | TCP | 8080 | 0.0.0.0/0 | Jenkins web UI |
| HTTP | TCP | 80 | 0.0.0.0/0 | Application access |

#### Step 1.5: Storage
- Root volume: **20 GB gp3** (sufficient for Jenkins + Docker)

#### Step 1.6: Launch Instance
Click **Launch Instance** and wait for creation.

#### Step 1.7: Verify EC2 Instance
**Path:** EC2 → Instances → Select `jenkins-server`
- Verify **Instance state: Running**
- Note **Public IPv4 address** (needed later)

---

### ✅ Step 1 Summary
By completing STEP 1, you now have:
- ✅ A running EC2 instance named `jenkins-server`
- ✅ Ubuntu Linux server ready for configuration
- ✅ Network access configured (SSH, Jenkins UI, HTTP)
- ✅ SSH key pair for secure access
- ✅ Foundation for the CI/CD pipeline

---

## STEP 2: IAM Role Setup for Jenkins (EC2 → ECR → ECS)

### Step 2 Overview
**What Are We Doing?**
Creating and attaching an IAM Role to the Jenkins EC2 instance so Jenkins can securely interact with AWS services without storing access keys.

**Jenkins will need permissions to:**
- Push Docker images to Amazon ECR
- Register and update ECS Task Definitions
- Deploy containers to ECS (Fargate)

### ⚠️ Important Concept
**We are NOT using AWS access keys inside Jenkins.**
Instead, we use **IAM Role + Temporary Credentials** – the industry standard for security.

---

### METHOD 1: GUI IMPLEMENTATION (AWS Console)

#### Step 2.1: Open IAM Role Creation Wizard
**Path:** AWS Management Console → IAM → Roles → Create role

#### Step 2.2: Select Trusted Entity
| Setting | Value | Why |
|---------|-------|-----|
| Trusted entity type | AWS service | EC2 needs to assume this role |
| Use case | EC2 | Jenkins runs on EC2 |

Click **Next**.

#### Step 2.3: Attach Permission Policies
Attach the following AWS managed policies:

| Policy Name | Purpose |
|-------------|---------|
| `AmazonEC2ContainerRegistryFullAccess` | Push Docker images to ECR |
| `AmazonECS_FullAccess` | Register task definitions & update services |

⚠️ *These are learning-friendly. In production, create custom least-privilege policies.*

Click **Next**.

#### Step 2.4: Name and Create Role
| Field | Value |
|-------|-------|
| Role name | `Jenkins-ECS-Role` |
| Description | IAM role for Jenkins EC2 to deploy to ECS |

Click **Create role**.

#### Step 2.5: Attach IAM Role to Jenkins EC2
**Path:** EC2 → Instances → Select `jenkins-server` → Actions → Security → Modify IAM role

| Field | Value |
|-------|-------|
| IAM role | `Jenkins-ECS-Role` |

Click **Update IAM role**.

#### Step 2.6: Verify IAM Role Attachment
**Path:** EC2 → Instances → `jenkins-server` → Scroll to **IAM role** section
- Verify `Jenkins-ECS-Role` is displayed

### ⚠️ Common Mistakes in Step 2
❌ Forgetting to attach role to EC2 instance  
❌ Using IAM user keys instead of roles  
❌ Giving `AdministratorAccess` (too permissive)  
❌ Attaching role to wrong instance  
❌ Skipping verification step  

---

### ✅ Step 2 Summary
By completing STEP 2, you now have:
- ✅ Secure IAM Role configured for Jenkins
- ✅ Jenkins authenticated without storing AWS keys
- ✅ ECR and ECS access enabled via IAM
- ✅ Production-style AWS security implemented
- ✅ Strong interview talking point (IAM roles vs access keys)

---

## STEP 3: Prepare EC2 Server for Jenkins & CI/CD

### Step 3 Overview
**What Are We Doing?**
Preparing the EC2 instance to become a DevOps-ready machine by:
- Securely connecting via SSH
- Updating the operating system
- Installing Docker
- Installing AWS CLI
- Verifying IAM role access

---

### PART A: CONNECT TO EC2 (SSH)

#### Step 3.1: Verify EC2 Is Reachable
**AWS Console Path:** EC2 → Instances → Select `jenkins-server`
- Confirm **Instance state: Running**
- Note **Public IPv4 address**
- Verify security group allows SSH (port 22) from your IP

#### Step 3.2: SSH into EC2
```bash
# Define variables
export EC2_USER=ubuntu
export EC2_IP=54.123.45.67  # Replace with your EC2 public IP
export KEY_FILE=jenkins-key.pem

# Set correct permissions for key file
chmod 400 $KEY_FILE

# SSH into EC2
ssh -i $KEY_FILE $EC2_USER@$EC2_IP
```

**✅ Success:** You're now logged into the EC2 server.

---

### PART B: SYSTEM UPDATE

#### Step 3.3: Update Operating System
```bash
sudo apt update && sudo apt upgrade -y
```

**Why:** Installs security patches and prevents dependency issues.

---

### PART C: INSTALL DOCKER

#### Step 3.4: Install Docker Engine
```bash
sudo apt install docker.io -y
```

#### Step 3.5: Start & Enable Docker
```bash
sudo systemctl start docker
sudo systemctl enable docker
```

#### Step 3.6: Verify Docker Installation
```bash
docker --version
```
**Expected:** `Docker version 24.0.x`

#### Step 3.7: Allow Non-Root Docker Access (Critical!)
By default, Docker requires `sudo`. Jenkins needs to run Docker without sudo.

```bash
# Add ubuntu user to docker group
sudo usermod -aG docker ubuntu

# Apply group changes without logout
newgrp docker

# Verify
docker ps
```
**If no permission error → Docker configured correctly.**

---

### PART D: INSTALL AWS CLI

#### Step 3.8: Install Dependencies
```bash
sudo apt install unzip -y
```

#### Step 3.9: Download AWS CLI v2
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o awscliv2.zip
```

#### Step 3.10: Install AWS CLI
```bash
unzip awscliv2.zip
sudo ./aws/install
```

#### Step 3.11: Verify AWS CLI Installation
```bash
aws --version
```
**Expected:** `aws-cli/2.x.x`

#### Step 3.12: Verify IAM Role Access (Critical Check!)
```bash
aws sts get-caller-identity
```
**Expected:** ARN containing `Jenkins-ECS-Role`

✅ **Confirms:** IAM role attached, no access keys required, AWS authentication working.

---

### ⚠️ Common Mistakes in Step 3
❌ Forgetting to update OS  
❌ Not adding user to docker group  
❌ Using `sudo docker` in pipelines (will fail)  
❌ Installing AWS CLI v1 instead of v2  
❌ Skipping IAM role verification  

---

### ✅ Step 3 Summary
By completing STEP 3, you now have:
- ✅ Secured EC2 server with latest updates
- ✅ Docker-enabled CI/CD host
- ✅ AWS CLI installed and configured
- ✅ IAM role verified (no access keys needed)
- ✅ DevOps-ready build machine

---

## STEP 4: Install & Configure Jenkins on EC2

### Step 4 Overview
**What Are We Doing?**
Installing Jenkins, configuring it correctly, and making it ready to run CI/CD pipelines.

**After this step:**
- Jenkins runs as a background service
- Jenkins UI is accessible via browser
- Jenkins can pull code from GitHub and build Docker images

---

### PART A: INSTALL JAVA (Jenkins Dependency)

#### Step 4.1: Install Java (OpenJDK 17)
```bash
sudo apt install openjdk-17-jdk -y
```

#### Step 4.2: Verify Java Installation
```bash
java -version
```
**Expected:** `openjdk version "17.x.x"`

---

### PART B: INSTALL JENKINS

#### Step 4.3: Add Jenkins Repository Key
```bash
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key \
  | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
```

#### Step 4.4: Add Jenkins Repository
```bash
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ \
  | sudo tee /etc/apt/sources.list.d/jenkins.list
```

#### Step 4.5: Install Jenkins
```bash
sudo apt update
sudo apt install jenkins -y
```

#### Step 4.6: Start & Enable Jenkins Service
```bash
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

#### Step 4.7: Verify Jenkins Service Status
```bash
sudo systemctl status jenkins
```
**Expected:** `active (running)`

---

### PART C: ACCESS JENKINS WEB UI

#### Step 4.8: Open Jenkins in Browser
```
http://<EC2_PUBLIC_IP>:8080
```
Example: `http://54.123.45.67:8080`

#### Step 4.9: Get Initial Admin Password
```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
Copy the displayed password.

#### Step 4.10: Unlock Jenkins
- Paste the password in browser
- Click **Continue**

#### Step 4.11: Install Plugins
Select **Install Suggested Plugins** (includes Git, Pipeline, Credentials)

#### Step 4.12: Create Admin User
Fill in:
- Username: `admin` (or your choice)
- Password: (create strong password)
- Full name: (optional)
- Email: (optional)

#### Step 4.13: Confirm Jenkins URL
Leave default (`http://<EC2_IP>:8080`) and click **Save and Finish**

🎉 **Jenkins setup complete!**

---

### PART D: INSTALL ADDITIONAL PLUGINS

#### Step 4.14: Install Required Plugins
**Path:** Jenkins Dashboard → Manage Jenkins → Plugins → Available Plugins

Search and install:

| Plugin | Purpose |
|--------|---------|
| **Docker Pipeline** | Build & push Docker images |
| **AWS Credentials** | AWS integration |
| **Pipeline: Stage View** | Visual pipeline view |

Restart Jenkins when prompted.

---

### PART E: CONFIGURE DOCKER FOR JENKINS USER

#### Step 4.15: Why This Is Needed
Jenkins runs as user `jenkins`. By default, `jenkins` cannot run Docker commands.

#### Step 4.16: Add Jenkins User to Docker Group
```bash
sudo usermod -aG docker jenkins
```

#### Step 4.17: Restart Jenkins
```bash
sudo systemctl restart jenkins
```

#### Step 4.18: Verify Docker Access
```bash
sudo su - jenkins
docker ps
exit
```
**If no error → Docker access configured correctly.**

---

### ⚠️ Common Mistakes in Step 4
❌ Forgetting to install Java first  
❌ Not opening port 8080 in security group  
❌ Not restarting Jenkins after Docker permission change  
❌ Skipping plugin installation  
❌ Using `sudo docker` in pipelines  

---

### ✅ Step 4 Summary
By completing STEP 4, you now have:
- ✅ Jenkins installed and running
- ✅ Jenkins UI accessible via browser
- ✅ Required plugins installed
- ✅ Docker access configured for Jenkins user
- ✅ Fully functional CI/CD server ready

---

## STEP 5: Clone Application Repository in Jenkins

### Step 5 Overview
**What Are We Doing?**
Connecting Jenkins with GitHub so Jenkins can access application source code and trigger pipelines automatically.

---

### PART A: PREPARE GITHUB REPOSITORY

#### Step 5.1: Verify Repository Structure
Your GitHub repository should contain:
```
my-app/
├── app.py / index.js / index.html  (application code)
├── Dockerfile                       (required for containerization)
└── README.md                        (optional)
```

**Sample Dockerfile (for Node.js app):**
```dockerfile
FROM node:14-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 80
CMD ["node", "app.js"]
```

**Sample Dockerfile (for Python app):**
```dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
EXPOSE 80
CMD ["python", "app.py"]
```

#### Step 5.2: Copy Repository URL
In GitHub:
- Open your repository
- Click **Code** button
- Copy **HTTPS URL** (e.g., `https://github.com/username/my-app.git`)

---

### PART B: CREATE JENKINS JOB (GUI)

#### Step 5.3: Open Jenkins Dashboard
```
http://<EC2_PUBLIC_IP>:8080
```
Login with admin credentials created in Step 4.

#### Step 5.4: Create New Jenkins Job
- Click **New Item**
- Enter name: `my-app-ci`
- Select **Pipeline**
- Click **OK**

#### Step 5.5: Configure Pipeline
Scroll to **Pipeline** section:

| Field | Value |
|-------|-------|
| Definition | **Pipeline script from SCM** |
| SCM | **Git** |
| Repository URL | Your GitHub HTTPS URL |
| Credentials | None (for public repos) |
| Branch Specifier | `*/main` or `*/master` |
| Script Path | `Jenkinsfile` |

Click **Save**.

---

### PART C: TEST CLONING

#### Step 5.6: Trigger First Build
- Open job: `my-app-ci`
- Click **Build Now**

#### Step 5.7: Verify Repository Clone
- Click the build number
- Click **Console Output**
- Look for:
  ```
  Cloning repository https://github.com/...
  Checking out revision ...
  ```

✅ **Confirms:** Jenkins successfully cloned the repository.

---

### PART D: VERIFY WORKSPACE (Optional)

#### Step 5.8: Check Workspace on EC2
SSH into EC2:
```bash
ls /var/lib/jenkins/workspace/
```
You should see `my-app-ci` directory.

Check files:
```bash
ls /var/lib/jenkins/workspace/my-app-ci/
```

---

### ⚠️ Common Mistakes in Step 5
❌ Wrong repository URL (typo)  
❌ Incorrect branch name  
❌ Missing Dockerfile in repository  
❌ Jenkins cannot access private repository (need credentials)  
❌ Wrong Jenkinsfile path  

---

### ✅ Step 5 Summary
By completing STEP 5, you now have:
- ✅ Jenkins connected to GitHub
- ✅ Source code cloning automated
- ✅ CI pipeline entry point created
- ✅ Jenkins workspace ready for Docker build

---

## STEP 6: Build Docker Image using Jenkins Pipeline

### Step 6 Overview
**What Are We Doing?**
Creating a Jenkins Pipeline that performs Continuous Integration (CI) by:
- Pulling latest code from GitHub
- Building a Docker image
- Tagging the image with build number and `latest`

---

### 📝 Complete Jenkinsfile

Create this `Jenkinsfile` in your GitHub repository root:

```groovy
pipeline {
  agent any

  environment {
    AWS_REGION   = 'ap-south-1'                    // Change to your region
    AWS_ACCOUNT_ID = '12345678'                 // Replace with your AWS account ID
    ECR_REPO_NAME = 'my-app-repo'                    // ECR repository name
    ECR_REPO_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO_NAME}"
    IMAGE_TAG    = "${BUILD_NUMBER}"
    ECS_CLUSTER  = 'my-cluster'
    ECS_SERVICE  = 'my-service'
    CONTAINER_NAME = 'my-container'
  }

  stages {
    stage('Checkout Code') {
      steps {
        git branch: 'main',
            url: 'https://github.com/username/my-app.git'   // Replace with your repo
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          sh '''
            echo "Building Docker image with tag: ${IMAGE_TAG}"
            docker build -t ${ECR_REPO_URI}:${IMAGE_TAG} .
            docker tag ${ECR_REPO_URI}:${IMAGE_TAG} ${ECR_REPO_URI}:latest
          '''
        }
      }
    }

    stage('Push Image to ECR') {
      steps {
        script {
          sh '''
            # Login to ECR
            aws ecr get-login-password --region ${AWS_REGION} | \
              docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
            
            # Push both tags
            docker push ${ECR_REPO_URI}:${IMAGE_TAG}
            docker push ${ECR_REPO_URI}:latest
          '''
        }
      }
    }

    stage('Deploy to ECS') {
      steps {
        script {
          sh '''
            # Force new deployment with latest image
            aws ecs update-service \
              --cluster ${ECS_CLUSTER} \
              --service ${ECS_SERVICE} \
              --force-new-deployment \
              --region ${AWS_REGION}
          '''
        }
      }
    }
  }

  post {
    success {
      echo 'Pipeline completed successfully!'
    }
    failure {
      echo 'Pipeline failed. Check logs for details.'
    }
  }
}
```

---

### 🔍 Detailed Explanation of Each Stage

#### Step 6.1: Pipeline Declaration
```groovy
pipeline {
  agent any
```
- **`pipeline`** – Declarative pipeline syntax
- **`agent any`** – Run on any available Jenkins executor (our EC2 instance)

#### Step 6.2: Environment Variables
```groovy
environment {
  AWS_REGION   = 'ap-south-1'
  AWS_ACCOUNT_ID = '12345678'
  ECR_REPO_NAME = 'my-app-repo'
  ECR_REPO_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO_NAME}"
  IMAGE_TAG    = "${BUILD_NUMBER}"
  ECS_CLUSTER  = 'my-cluster'
  ECS_SERVICE  = 'my-service'
}
```
**Why environment block?**
- Avoids hardcoding values
- Makes pipeline reusable
- Industry best practice

**Key variables:**
- `IMAGE_TAG = "${BUILD_NUMBER}"` – Uses Jenkins build number for versioning
- `ECR_REPO_URI` – Full URI for ECR repository

#### Step 6.3: Stage – Checkout Code
```groovy
stage('Checkout Code') {
  steps {
    git branch: 'main',
        url: 'https://github.com/username/my-app.git'
  }
}
```
**What happens:**
- Jenkins contacts GitHub
- Pulls latest code from `main` branch
- Copies code to Jenkins workspace

#### Step 6.4: Stage – Build Docker Image (Core CI Step)
```groovy
stage('Build Docker Image') {
  steps {
    script {
      sh '''
        docker build -t ${ECR_REPO_URI}:${IMAGE_TAG} .
        docker tag ${ECR_REPO_URI}:${IMAGE_TAG} ${ECR_REPO_URI}:latest
      '''
    }
  }
}
```

**Command breakdown:**
1. `docker build -t ${ECR_REPO_URI}:${IMAGE_TAG} .`
   - Reads Dockerfile from current directory
   - Builds Docker image
   - Tags with ECR URI + build number (e.g., `:15`)

2. `docker tag ... :latest`
   - Creates additional `latest` tag
   - Useful for default deployments

**Why tag with build number?**
- Every build gets unique version
- Enables easy rollbacks
- Full traceability

---

### ✅ Verification Steps

#### Step 6.5: Verify via Jenkins Console
After build completes:
- Click build number → **Console Output**
- Look for:
  ```
  Successfully built abcdef123456
  Successfully tagged 123456789012.dkr.ecr.ap-south-1.amazonaws.com/my-app-repo:1
  ```

#### Step 6.6: Verify Locally on EC2
SSH into EC2 and run:
```bash
docker images
```
**Expected output:**
```
REPOSITORY                                                        TAG       IMAGE ID
123456789012.dkr.ecr.ap-south-1.amazonaws.com/my-app-repo        1         abcdef123456
123456789012.dkr.ecr.ap-south-1.amazonaws.com/my-app-repo        latest    abcdef123456
```

---

### ⚠️ Common Mistakes in Step 6
❌ **Dockerfile missing** in repository  
❌ **Jenkins user not in docker group** (permission denied)  
❌ Using `sudo docker` in pipeline  
❌ Hardcoding image tags (should use `${BUILD_NUMBER}`)  
❌ Missing `script` block around `sh` commands  
❌ Incorrect ECR repository URI format  

---

### ✅ Step 6 Summary
By completing STEP 6, you now have:
- ✅ Jenkins pulling latest code automatically
- ✅ Docker image built on every run
- ✅ Image versioned using build numbers
- ✅ `latest` tag maintained
- ✅ CI pipeline producing deployable artifacts

---

## STEP 7: Push Docker Image to Amazon ECR

### Step 7 Overview
**What Are We Doing?**
Jenkins will securely authenticate with Amazon ECR and push the Docker image built in STEP 6 to a private container registry, making it available for ECS deployment.

---

### PART A: CREATE ECR REPOSITORY (GUI)

#### Step 7.1: Open ECR Console
**Path:** AWS Management Console → Amazon ECR → Repositories

#### Step 7.2: Create ECR Repository
Click **Create repository** and configure:

| Field | Value |
|-------|-------|
| Repository name | `my-app-repo` |
| Visibility | **Private** |
| Tag immutability | Disabled (for learning) |
| Encryption | AES-256 (default) |

Click **Create repository**.

#### Step 7.3: Note Repository URI
After creation, note the URI:
```
123456789012.dkr.ecr.ap-south-1.amazonaws.com/my-app-repo
```

---

### PART B: PUSH IMAGE USING JENKINS PIPELINE

#### Step 7.4: Jenkins Pipeline – Push Stage
This stage is already in your Jenkinsfile from STEP 6:

```groovy
stage('Push Image to ECR') {
  steps {
    script {
      sh '''
        # Login to ECR (generates temporary token)
        aws ecr get-login-password --region ${AWS_REGION} | \
          docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
        
        # Push versioned image
        docker push ${ECR_REPO_URI}:${IMAGE_TAG}
        
        # Push latest tag
        docker push ${ECR_REPO_URI}:latest
      '''
    }
  }
}
```

#### Step 7.5: Line-by-Line Explanation

**🔹 ECR Login:**
```bash
aws ecr get-login-password --region ${AWS_REGION} | \
  docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
```
- `aws ecr get-login-password` – Requests temporary authentication token using IAM role
- Token passed securely to `docker login` via stdin
- No passwords stored or exposed

**🔹 Push Commands:**
```bash
docker push ${ECR_REPO_URI}:${IMAGE_TAG}
docker push ${ECR_REPO_URI}:latest
```
- Pushes both versioned and latest tags
- Versioned tag enables rollbacks
- Latest tag used for default deployments

---

### PART C: VERIFICATION

#### Step 7.6: Verify in AWS Console (GUI)
**Path:** Amazon ECR → Repositories → `my-app-repo`

You should see:
| Tag | Pushed At |
|-----|-----------|
| `latest` | Today |
| `1` (or your build number) | Today |

#### Step 7.7: Verify via CLI (Optional)
```bash
export AWS_REGION=ap-south-1
export REPO_NAME=my-app-repo

aws ecr list-images \
  --repository-name $REPO_NAME \
  --region $AWS_REGION
```

**Expected output:**
```json
{
  "imageIds": [
    {"imageTag": "latest", ...},
    {"imageTag": "1", ...}
  ]
}
```

---

### ⚠️ Common ECR Errors & Fixes

| Error | Cause | Fix |
|-------|-------|-----|
| `no basic auth credentials` | Not logged into ECR | Check login command |
| `AccessDeniedException` | IAM role missing permissions | Attach ECR policy to role |
| `repository does not exist` | Repository name mismatch | Verify repository URI |
| `denied: requested access` | Wrong account/region | Check AWS variables |

---

### ✅ Step 7 Summary
By completing STEP 7, you now have:
- ✅ Secure Amazon ECR repository created
- ✅ Jenkins authenticated without access keys
- ✅ Docker images pushed automatically
- ✅ Versioned (`:1`, `:2`) and `latest` tags stored
- ✅ Artifact ready for deployment

---

## STEP 8: Create ECS Cluster (AWS Fargate)

### Step 8 Overview
**What Are We Doing?**
Creating an Amazon ECS Cluster that will act as a logical boundary for running containers on AWS Fargate (serverless).

---

### METHOD 1: GUI IMPLEMENTATION

#### Step 8.1: Open ECS Console
**Path:** AWS Management Console → Amazon ECS

#### Step 8.2: Create ECS Cluster
- Click **Clusters** in left navigation
- Click **Create cluster**

#### Step 8.3: Configure Cluster Settings

| Setting | Value | Why |
|---------|-------|-----|
| Cluster template | **AWS Fargate** | Serverless, no EC2 management |
| Cluster name | `my-cluster` | Descriptive name |
| Networking | Default VPC | Use existing network |

#### Step 8.4: Create Cluster
Click **Create**. AWS provisions the cluster (almost instantly).

#### Step 8.5: Verify Cluster
**Path:** ECS → Clusters → `my-cluster`
- Status: `ACTIVE`
- Services: `0`
- Tasks: `0` (expected at this stage)

---

### ⚠️ Common Mistakes
❌ Confusing cluster with service  
❌ Expecting containers to run immediately  
❌ Thinking EC2 instances should appear  
❌ Using EC2 launch type unnecessarily  

---

### ✅ Step 8 Summary
By completing STEP 8, you now have:
- ✅ ECS cluster created
- ✅ Fargate-based runtime ready
- ✅ No servers to manage
- ✅ Foundation for container deployment

---

## STEP 9: Create ECS Task Definition (Fargate)

### Step 9 Overview
**What Are We Doing?**
Creating an ECS Task Definition – a blueprint that tells ECS:
- Which Docker image to run
- CPU and memory allocation
- Ports to expose
- IAM execution role
- Logging configuration

---

### METHOD 1: GUI IMPLEMENTATION

#### Step 9.1: Open Task Definition Wizard
**Path:** Amazon ECS → Task Definitions → Create new task definition

#### Step 9.2: Select Launch Type
Select: **AWS Fargate** → Click **Next**

#### Step 9.3: Configure Task Definition Basics

| Field | Value |
|-------|-------|
| Task definition family | `my-task` |
| Operating system | Linux |
| Task role | (Leave empty for now) |
| Task execution role | **ecsTaskExecutionRole** |

**Why execution role is mandatory:**
- Allows ECS to pull images from ECR
- Enables CloudWatch logging

#### Step 9.4: Configure Task Size
Select valid Fargate combination:

| CPU | Memory |
|-----|--------|
| **0.25 vCPU** | **0.5 GB** |

*Suitable for lightweight web applications*

#### Step 9.5: Add Container Definition
Click **Add container**:

| Field | Value |
|-------|-------|
| Container name | `my-container` |
| Image URI | `<ECR_REPO_URI>:latest` (from Step 7) |
| Essential | ✅ Yes |

**Port mappings:**
| Setting | Value |
|---------|-------|
| Container port | `80` |
| Protocol | TCP |

**Logging (recommended):**
| Setting | Value |
|---------|-------|
| Log driver | `awslogs` |
| Log group | `/ecs/my-task` |
| Region | Your region |
| Stream prefix | `ecs` |

Click **Create**.

#### Step 9.6: Verify Task Definition
**Path:** ECS → Task Definitions → `my-task`
- Revision number visible (e.g., `:1`)
- Status: `ACTIVE`
---

### ⚠️ Common Mistakes
❌ Forgetting execution role (containers won't start)  
❌ Using wrong image URI  
❌ Invalid CPU–memory combination  
❌ Wrong port mapping (should match application)  
❌ Expecting task to run automatically  

---

### ✅ Step 9 Summary
By completing STEP 9, you now have:
- ✅ ECS task blueprint created
- ✅ Amazon ECR image referenced
- ✅ IAM execution role attached
- ✅ Logging configured
- ✅ Ready for ECS service creation

---

## STEP 10: Create ECS Service (AWS Fargate)

### Step 10 Overview
**What Are We Doing?**
Creating an ECS Service that:
- Runs containers using the Task Definition from STEP 9
- Maintains desired number of containers
- Auto-heals failed containers
- Forms foundation for load balancing

---

### METHOD 1: GUI IMPLEMENTATION

#### Step 10.1: Open ECS Cluster
**Path:** Amazon ECS → Clusters → `my-cluster`

#### Step 10.2: Create Service
- Click **Services** tab
- Click **Create**

#### Step 10.3: Configure Service Basics

| Field | Value |
|-------|-------|
| Launch type | **Fargate** |
| Task Definition Family | `my-task` |
| Revision | **Latest** |
| Service name | `my-service` |
| Desired tasks | `1` |

#### Step 10.4: Networking Configuration
| Field | Value |
|-------|-------|
| VPC | Same as cluster |
| Subnets | Select at least one subnet |
| Security group | Create new or select existing |
| Assign public IP | **ENABLED** (for testing) |

**Security group rule (temporary):**
- Type: HTTP
- Port: 80
- Source: `0.0.0.0/0`

#### Step 10.5: Load Balancer (Skip for now)
Select: **No load balancer** (will add in next steps)

Click **Create service**.

#### Step 10.6: Verify Service
**Path:** ECS → Clusters → `my-cluster` → Services → `my-service`
- Desired tasks: `1`
- Running tasks: `1`
- Status: `ACTIVE`

#### Step 10.7: Verify Running Task
Click the service → **Tasks** tab
- Task status: `RUNNING`
- Container status: `RUNNING`

✅ **Confirms:** Image pulled from ECR, container running on Fargate

---

### 🔍 What ECS Did Internally
- Read task definition
- Pulled image from ECR
- Requested compute from Fargate
- Created Elastic Network Interface (ENI)
- Started container
- Began monitoring health

---

### ⚠️ Common Mistakes
❌ Expecting ALB to be attached automatically  
❌ Selecting wrong subnets (different AZ)  
❌ Security group blocking traffic  
❌ Accidentally using EC2 launch type  
❌ Forgetting desired count  

---

### ✅ Step 10 Summary
By completing STEP 10, you now have:
- ✅ ECS Service created
- ✅ Container running on Fargate
- ✅ Auto-healing enabled
- ✅ Application running (not yet public)

---

## STEP 11: Create Application Load Balancer (ALB)

### Step 11 Overview
**What Are We Doing?**
Creating an Application Load Balancer that will:
- Expose the application publicly
- Distribute traffic to ECS tasks
- Perform health checks
- Enable zero-downtime deployments

---

### METHOD 1: GUI IMPLEMENTATION

#### Step 11.1: Open Load Balancer Console
**Path:** AWS Management Console → EC2 → Load Balancers

#### Step 11.2: Create Load Balancer
- Click **Create Load Balancer**
- Select **Application Load Balancer**
- Click **Create**

#### Step 11.3: Configure Load Balancer Basics

| Field | Value |
|-------|-------|
| Name | `my-alb` |
| Scheme | **Internet-facing** |
| IP address type | `ipv4` |

#### Step 11.4: Network Mapping
| Field | Value |
|-------|-------|
| VPC | Same as ECS cluster |
| Subnets | Select **two subnets** (for HA) |

#### Step 11.5: Security Group
Create new security group:

| Rule | Port | Source |
|------|------|--------|
| HTTP | 80 | `0.0.0.0/0` |

#### Step 11.6: Listeners and Routing
| Field | Value |
|-------|-------|
| Listener protocol | HTTP |
| Listener port | 80 |
| Default action | **Create target group** (Step 12) |

Click **Create load balancer**.

#### Step 11.7: Verify ALB
**Path:** EC2 → Load Balancers → `my-alb`
- State: `active`
- DNS name: `my-alb-123456.elb.amazonaws.com`

---

### ⚠️ Common Mistakes
❌ Using Network Load Balancer instead of ALB  
❌ Selecting only one subnet (not HA)  
❌ Attaching ALB security group to ECS tasks  
❌ Expecting app to work before target group  

---

### ✅ Step 11 Summary
By completing STEP 11, you now have:
- ✅ Application Load Balancer created
- ✅ Public DNS endpoint available
- ✅ Listener configured on port 80
- ✅ Foundation for traffic routing

---

## STEP 12: Create Target Group for ECS Service

### Step 12 Overview
**What Are We Doing?**
Creating a Target Group that acts as the bridge between ALB and ECS tasks, defining where traffic should be forwarded and how health checks work.

---

### METHOD 1: GUI IMPLEMENTATION

#### Step 12.1: Open Target Group Creation
**Path:** EC2 → Target Groups → Create target group

#### Step 12.2: Choose Target Type

| Field | Value | Why |
|-------|-------|-----|
| Target type | **IP addresses** | Fargate uses IP-based targets |

#### Step 12.3: Configure Basic Settings

| Field | Value |
|-------|-------|
| Target group name | `my-target-group` |
| Protocol | HTTP |
| Port | 80 |
| VPC | Same as ECS cluster |

#### Step 12.4: Configure Health Checks (Critical!)

| Setting | Value |
|---------|-------|
| Health check protocol | HTTP |
| Health check path | `/` |
| Port | Traffic port |
| Healthy threshold | 5 |
| Unhealthy threshold | 2 |
| Timeout | 5 seconds |
| Interval | 30 seconds |

#### Step 12.5: Register Targets (SKIP)
When asked to register targets:
- Click **Next without adding targets**
- **Why?** ECS will register tasks automatically

#### Step 12.6: Create Target Group
Click **Create target group**.

#### Step 12.7: Verify Target Group
**Path:** EC2 → Target Groups → `my-target-group`
- Targets: `0` (expected – not yet attached)
- Health checks: Configured

---

### ⚠️ Common Mistakes
❌ Choosing "Instance" instead of "IP" for target type  
❌ Wrong VPC selection  
❌ Incorrect health check path (should match your app)  
❌ Manually registering targets for ECS  

---

### ✅ Step 12 Summary
By completing STEP 12, you now have:
- ✅ Target group created
- ✅ Health checks configured
- ✅ ALB ready to forward traffic
- ✅ Waiting for ECS service attachment

---

## STEP 13: Attach Target Group to ECS Service

### Step 13 Overview
**What Are We Doing?**
Connecting all pieces together by attaching the Target Group to the ECS Service, enabling traffic flow from ALB to containers.

---

### METHOD 1: GUI IMPLEMENTATION

#### Step 13.1: Open ECS Service
**Path:** Amazon ECS → Clusters → `my-cluster` → Services → `my-service`

#### Step 13.2: Update Service
Click **Update**

#### Step 13.3: Configure Load Balancer Integration
Navigate to **Load balancing** section:

| Field | Value |
|-------|-------|
| Load balancer type | **Application Load Balancer** |
| Load balancer | `my-alb` |
| Listener | **HTTP : 80** |
| Target group | `my-target-group` |

**Container mapping:**
| Field | Value |
|-------|-------|
| Container name | `my-container` |
| Container port | `80` |

#### Step 13.4: Complete Service Update
- Skip remaining sections (keep defaults)
- Click **Update service**
- ECS begins a new deployment

#### Step 13.5: Monitor Deployment
**Path:** ECS → Services → `my-service`
Watch:
- Deployment status
- Running tasks

#### Step 13.6: Verify Target Registration
**Path:** EC2 → Target Groups → `my-target-group` → **Targets**

You should see:
| Target (IP) | Status |
|-------------|--------|
| 10.0.1.45 | `healthy` |

#### Step 13.7: Final Test – Access Application
Open browser and visit:
```
http://<ALB_DNS_NAME>
```
**Example:** `http://my-alb-123456789.elb.amazonaws.com`

🎉 **Your application should load successfully!**


---

### 🔍 What AWS Does Internally
- Updates service configuration
- Launches new tasks
- Registers task ENIs with target group
- ALB performs health checks
- Traffic routes only to healthy tasks
- **Zero-downtime deployment achieved**

---

### 🚨 Troubleshooting (If App Does Not Load)

| Issue | Cause | Fix |
|-------|-------|-----|
| **Target unhealthy** | Wrong port or health path | Check container port & health check path |
| **503 error** | No healthy targets | Wait for health checks or fix application |
| **Timeout** | Security group blocking | Allow ALB → ECS traffic |
| **Page not found** | App not listening on port 80 | Update container/app config |
| **Connection refused** | Container not running | Check ECS service logs |

---

### ✅ Step 13 Summary
By completing STEP 13, you now have:
- ✅ Target group attached to ECS service
- ✅ ALB routing traffic to containers
- ✅ Health checks passing
- ✅ Application publicly accessible
- ✅ **End-to-end CI/CD pipeline completed!**

---

## 12. Monitoring & Observability (Production Readiness)

A CI/CD pipeline is incomplete without visibility into **what happens after deployment**.  
This project includes **production-grade monitoring and logging** to detect deployment failures and runtime issues early.

### What Is Covered
- Centralized ECS container logging using **CloudWatch Logs**
- ECS service monitoring for:
  - Task crashes
  - Failed or unstable deployments
  - High memory utilization
- Application Load Balancer (ALB) monitoring for:
  - Unhealthy targets
  - HTTP 5XX application errors
- Real-time alerting using **Amazon SNS**
- Failure simulation to validate alarms and alerts

### Why This Matters
CI pipelines (like Jenkins) can report success even when an application fails at runtime.  
This monitoring layer ensures:
- Failures are detected automatically
- Alerts reach engineers immediately
- Downtime is minimized
- Root cause analysis is possible using logs

📄 **Detailed monitoring design, alarm configuration, and validation steps:**  
[Automated Monitoring & Logging for Deployment Issues](./docs/monitoring-and-alerting.md)

## 13. Final Takeaway

This project demonstrates a **complete, production-ready CI/CD pipeline** using Jenkins, Docker, AWS ECS (Fargate), and an Application Load Balancer.

Beyond automated build and deployment, it also includes:
- Deployment validation through ECS and ALB
- Centralized logging for runtime visibility
- Automated monitoring and real-time alerting
- Failure simulation to validate system reliability

Together, these components reflect how modern cloud-native applications are built, deployed, monitored, and operated in real-world environments.

---

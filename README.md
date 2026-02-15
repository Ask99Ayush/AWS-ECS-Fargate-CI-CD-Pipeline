# Automated CI/CD Pipeline with Jenkins, Docker, AWS ECS (Fargate) & Application Load Balancer
---

## Overview

This repository documents a **production-grade CI/CD pipeline** that automatically builds, packages, and deploys a containerized application to **AWS ECS (Fargate)** using **Jenkins**, **Docker**, **Amazon ECR**, and an **Application Load Balancer (ALB)**.

Whenever code is pushed to GitHub, the pipeline:
- Builds the application
- Creates a Docker image
- Pushes the image to Amazon ECR
- Deploys it to AWS ECS (Fargate)
- Exposes it via a stable, load-balanced endpoint

This architecture mirrors real-world DevOps practices used by startups, SaaS companies, and large enterprises.

---

## Table of Contents

- [Overview](#overview)
- [Problem Statement](#problem-statement)
- [Understanding CI/CD](#understanding-cicd)
- [What This Project Solves](#what-this-project-solves)
- [Benefits](#benefits)
- [Tools & Technologies](#tools--technologies)
- [High-Level Architecture](#high-level-architecture)
- [Workflow / Data Flow](#workflow--data-flow)
- [Deployment Strategies & Zero Downtime](#deployment-strategies--zero-downtime)
- [Security Best Practices](#security-best-practices)
- [Implementation Guide](#implementation-guide)
  - [Step 1: EC2 for Jenkins](#step-1-ec2-for-jenkins)
  - [Step 2: IAM Role for Jenkins](#step-2-iam-role-for-jenkins)
  - [Step 3: Prepare EC2 Server](#step-3-prepare-ec2-server)
  - [Step 4: Install & Configure Jenkins](#step-4-install--configure-jenkins)
  - [Step 5: Clone GitHub Repository](#step-5-clone-github-repository)
  - [Step 6: Build Docker Image (CI)](#step-6-build-docker-image-ci)
  - [Step 7: Push Image to Amazon ECR](#step-7-push-image-to-amazon-ecr)
  - [Step 8: Create ECS Cluster (Fargate)](#step-8-create-ecs-cluster-fargate)
  - [Step 9: Create ECS Task Definition](#step-9-create-ecs-task-definition)
  - [Step 10: Create ECS Service](#step-10-create-ecs-service)
  - [Step 11: Create Application Load Balancer](#step-11-create-application-load-balancer)
  - [Step 12: Create Target Group](#step-12-create-target-group)
  - [Step 13: Attach Target Group to ECS Service](#step-13-attach-target-group-to-ecs-service)
- [Troubleshooting](#troubleshooting)
- [Final Takeaway](#final-takeaway)

---

## Problem Statement

### Traditional Deployment Challenges

Manual deployment workflows typically involve:
- Manual builds
- Manual Docker image creation
- Manual image pushes
- Manual ECS updates
- Manual service restarts

This approach introduces:
- Human error
- Inconsistent environments
- Poor scalability
- High operational risk
- No safe rollback mechanism

### Why Automation Is Mandatory

Modern applications:
- Change frequently
- Require 24/7 availability
- Must scale reliably

CI/CD automation ensures **speed, safety, repeatability, and reliability**.

---

## Understanding CI/CD

### Continuous Integration (CI)

CI automatically:
- Pulls code from GitHub
- Builds the application
- Produces deployable artifacts

### Continuous Deployment (CD)

CD automatically:
- Deploys verified artifacts to production
- Updates running services without manual intervention

### CI vs CD Models

| Term                    | Description                                |
|-------------------------|--------------------------------------------|
| Continuous Integration | Build and validate code                    |
| Continuous Delivery    | Ready for deployment (manual approval)     |
| Continuous Deployment  | Automatic deployment to production         |

This project implements **Continuous Deployment**.

---

## What This Project Solves

- Eliminates manual deployment steps
- Uses containerization for consistency
- Runs on serverless infrastructure
- Provides a stable public endpoint
- Implements industry-standard DevOps workflows

---

## Benefits

- Fully automated CI/CD pipeline
- Versioned and traceable Docker images
- Zero-downtime deployments (rolling updates)
- Secure, private container registry
- Serverless container execution
- Scalable and production-aligned architecture

---

## Tools & Technologies

| Tool | Purpose |
|-----|--------|
| GitHub | Source code management & pipeline trigger |
| Jenkins | CI/CD orchestration engine |
| Docker | Containerization & immutable builds |
| Amazon ECR | Secure private Docker registry |
| Amazon ECS (Fargate) | Serverless container orchestration |
| Application Load Balancer | Traffic routing & health checks |

---

## Architecture Diagram
  ![image](https://github.com/user-attachments/assets/612b1b18-7636-4d2b-aff6-cf5e971c2797)

---

## Workflow / Data Flow

1. Developer pushes code to GitHub
2. GitHub webhook triggers Jenkins
3. Jenkins clones repository
4. Docker image is built and tagged
5. Image is pushed to Amazon ECR
6. ECS service is updated
7. Fargate launches new containers
8. ALB routes traffic to healthy tasks

---

## Deployment Strategies & Zero Downtime

### Rolling Deployment
- Default ECS behavior
- Gradual replacement of tasks

### Blue-Green Deployment
- Parallel environments
- Safer but more complex

### Zero-Downtime Mechanism
- Parallel task execution
- ALB health checks
- Traffic routing to healthy targets only

---

## Security Best Practices

### Recommended Practices
- IAM roles with least privilege
- No hardcoded AWS credentials
- Private ECR repositories
- Jenkins credentials store
- ECS task execution roles

### Common Mistakes
- Using AWS access keys in pipelines
- Overusing `latest` without versioned tags
- Skipping health checks
- Manual ECS restarts

---

## Implementation Guide

### Step 1: EC2 for Jenkins
- Launch Ubuntu 22.04 LTS EC2 instance
- Open ports: 22 (SSH), 8080 (Jenkins), 80 (HTTP)
- Use `t2.micro` or `t3.micro`

### Step 2: IAM Role for Jenkins
- Create IAM role for EC2
- Attach ECR and ECS permissions
- Avoid IAM user access keys
- Verify using:
```bash
aws sts get-caller-identity
````

### Step 3: Prepare EC2 Server

* Update OS
* Install Docker
* Add user to docker group
* Install AWS CLI v2

### Step 4: Install & Configure Jenkins

* Install OpenJDK 17
* Install Jenkins from official repository
* Install required plugins
* Grant Docker access to Jenkins user

### Step 5: Clone GitHub Repository

* Create Jenkins Pipeline job
* Configure Git SCM
* Verify repository cloning

### Step 6: Build Docker Image (CI)

* Use Jenkinsfile
* Build image with build number tags
* Maintain `latest` tag

### Step 7: Push Image to Amazon ECR

* Authenticate using IAM role
* Push versioned and latest images
* Verify in ECR console

### Step 8: Create ECS Cluster (Fargate)

* Create ECS cluster with Fargate launch type
* No EC2 instances required

### Step 9: Create ECS Task Definition

* Define CPU, memory, ports
* Configure logging
* Attach execution role
* Reference ECR image

### Step 10: Create ECS Service

* Use Fargate launch type
* Desired count ≥ 1
* Enable auto-healing

### Step 11: Create Application Load Balancer

* Internet-facing ALB
* Listener on port 80
* Multi-AZ subnets

### Step 12: Create Target Group

* Target type: IP
* Configure health checks
* Do not manually register targets

### Step 13: Attach Target Group to ECS Service

* Attach ALB to ECS service
* Map container port
* Verify healthy targets
* Access application via ALB DNS

---

## Troubleshooting

| Issue            | Cause                | Resolution              |
| ---------------- | -------------------- | ----------------------- |
| 503 Error        | No healthy targets   | Fix health checks       |
| Target unhealthy | Wrong port/path      | Verify container config |
| Timeout          | Security group issue | Allow ALB → ECS         |
| App not loading  | App not on port 80   | Update container        |

---

## Final Takeaway

By completing this project, you gain:

* Practical understanding of CI/CD pipelines
* Real-world AWS container deployment experience
* Strong DevOps and cloud fundamentals
* Interview-ready architecture knowledge
* Production-aligned implementation skills

This project represents a **complete, enterprise-style CI/CD workflow** suitable for professional portfolios and technical interviews.

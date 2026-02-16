# Automated CI/CD Pipeline using Jenkins, Docker, AWS ECS (Fargate) & Application Load Balancer

## Complete Production-Grade Implementation Guide

<p align="center">
  <img src="./docs/architecture_diagram/architecture_diagram.png" alt="CI/CD Architecture Diagram" width="85%">
</p>

<p align="center">
  <strong>End-to-end automated CI/CD pipeline with zero-downtime deployments</strong><br>
  From GitHub commit to live production on AWS
</p>

---

## Project Overview

This repository demonstrates a **real-world, production-grade CI/CD pipeline** that automates the complete lifecycle of deploying a containerized application on AWS.

Whenever code is pushed to GitHub, the pipeline automatically:

* Builds the application
* Creates a Docker image
* Pushes the image to Amazon ECR
* Deploys the application to AWS ECS (Fargate)
* Exposes it via an Application Load Balancer

Once configured, the process requires **no manual intervention**.

---

## Key Capabilities

* Fully automated Continuous Integration and Continuous Deployment
* Consistent builds using Docker
* Serverless container execution with AWS Fargate
* Zero-downtime deployments using Application Load Balancer
* Secure IAM role–based AWS access
* Centralized logging, monitoring, and alerting
* Architecture aligned with real production systems

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Key Capabilities](#key-capabilities)
3. [Introduction](#1-introduction)

   * [What This Project Demonstrates](#what-this-project-demonstrates)
   * [Real-World Mapping](#real-world-mapping)
   * [Industry Relevance](#industry-relevance)
4. [Why This Project Is Needed](#2-why-this-project-is-needed)
5. [Understanding CI/CD](#3-understanding-cicd)
6. [What This Project Solves](#4-what-this-project-solves)
7. [Benefits](#5-benefits)
8. [Tools & Technologies](#6-tools--technologies)
9. [High-Level Architecture](#7-high-level-architecture)
10. [Internal Data & Control Flow](#8-internal-data--control-flow)
11. [Deployment Strategies & Zero Downtime](#9-deployment-strategies--zero-downtime)
12. [Security Best Practices & Common Mistakes](#10-security-best-practices--common-mistakes)
13. [Step-by-Step Implementation Guide](#11-step-by-step-implementation-guide)

    * [STEP 1 – Create EC2 Instance for Jenkins](#step-1--create-ec2-instance-for-jenkins)
    * [STEP 2 – IAM Role Setup](#step-2--iam-role-setup)
    * [STEP 3 – Prepare EC2 Server](#step-3--prepare-ec2-server)
    * [STEP 4 – Install and Configure Jenkins](#step-4--install-and-configure-jenkins)
    * [STEP 5 – Connect Jenkins with GitHub](#step-5--connect-jenkins-with-github)
    * [STEP 6 – Build Docker Image (CI)](#step-6--build-docker-image-ci)
    * [STEP 7 – Push Docker Image to Amazon ECR](#step-7--push-docker-image-to-amazon-ecr)
    * [STEP 8 – Create ECS Cluster](#step-8--create-ecs-cluster)
    * [STEP 9 – Create ECS Task Definition](#step-9--create-ecs-task-definition)
    * [STEP 10 – Create ECS Service](#step-10--create-ecs-service)
    * [STEP 11 – Create Application Load Balancer](#step-11--create-application-load-balancer)
    * [STEP 12 – Create Target Group](#step-12--create-target-group)
    * [STEP 13 – Attach Target Group to ECS Service](#step-13--attach-target-group-to-ecs-service)
14. [Monitoring & Observability](#12-monitoring--observability)
15. [Final Takeaway](#13-final-takeaway)

---

## 1. Introduction

### What This Project Demonstrates

This project shows how modern DevOps teams design and operate CI/CD pipelines for reliable, repeatable, and safe production deployments. The workflow and architecture mirror what is commonly used in startups, SaaS platforms, and enterprise environments.

---

### Real-World Mapping

| Component                 | Responsibility                                |
| ------------------------- | --------------------------------------------- |
| GitHub                    | Source code management and pipeline trigger   |
| Jenkins                   | CI/CD orchestration engine                    |
| Docker                    | Application packaging and runtime consistency |
| Amazon ECR                | Secure container image registry               |
| AWS ECS (Fargate)         | Serverless container orchestration            |
| Application Load Balancer | Traffic routing and health checks             |

---

### Industry Relevance

This architecture is widely used to:

* Deploy microservices
* Release features continuously
* Reduce deployment risk
* Achieve zero-downtime updates

Understanding this project means understanding **how real production deployments work**.

---

## 2. Why This Project Is Needed

### Problems with Manual Deployment

Manual deployments are:

* Time-consuming
* Error-prone
* Inconsistent across team members
* Difficult to scale
* Risky in production environments

### Why Automation Is Required

Automation provides:

* Speed and repeatability
* Predictable deployment behavior
* Safer rollbacks
* Higher system reliability
* Confidence in production releases

---

## 3. Understanding CI/CD

| Term                   | Description                     | Manual Approval |
| ---------------------- | ------------------------------- | --------------- |
| Continuous Integration | Automatic build and validation  | No              |
| Continuous Delivery    | Deployment-ready artifacts      | Yes             |
| Continuous Deployment  | Automatic production deployment | No              |

This project implements **full Continuous Deployment**.

---

## 4. What This Project Solves

* Eliminates manual build and deployment steps
* Ensures environment consistency using containers
* Uses serverless compute to reduce operational overhead
* Provides a stable public endpoint via ALB
* Demonstrates real DevOps workflows used in industry

---

## 5. Benefits

* Automated deployments on every code change
* Zero-downtime application updates
* Scalable container-based architecture
* Secure image storage and access
* Strong foundation for DevOps interviews
* Reusable design for real-world projects

---

## 6. Tools & Technologies

| Tool                      | Purpose                        |
| ------------------------- | ------------------------------ |
| GitHub                    | Source control and CI trigger  |
| Jenkins                   | Pipeline orchestration         |
| Docker                    | Containerization               |
| Amazon ECR                | Docker image registry          |
| AWS ECS (Fargate)         | Serverless container execution |
| Application Load Balancer | Traffic distribution           |

---

## 7. High-Level Architecture

<p align="center">
  <img src="./docs/architecture_diagram/architecture_diagram.png" width="90%">
</p>

---

## 8. Internal Data & Control Flow

1. Developer pushes code to GitHub
2. GitHub webhook triggers Jenkins
3. Jenkins pulls source code
4. Docker image is built
5. Image is pushed to Amazon ECR
6. ECS service is updated
7. Fargate launches new containers
8. ALB performs health checks
9. Traffic is routed to healthy containers

---

## 9. Deployment Strategies & Zero Downtime

### Rolling Deployments

* Default ECS deployment strategy
* Gradual replacement of running containers
* No service interruption

### Zero-Downtime Mechanism

* Parallel task execution
* Health-check-based traffic routing
* Automatic traffic shifting
* No single point of failure

---

## 10. Security Best Practices & Common Mistakes

### Implemented Best Practices

* IAM roles with least privilege
* No AWS credentials stored in Jenkins
* Private Amazon ECR repositories
* Secure Jenkins credential handling
* Separate security groups for ALB and ECS

### Common Mistakes Avoided

* Hardcoded AWS access keys
* Using only the `latest` image tag
* Manual ECS restarts
* Over-permissive IAM policies

---

## 11. Step-by-Step Implementation Guide


### STEP 1 – Create EC2 Instance for Jenkins

Ubuntu-based EC2 instance with required network access and storage.

### STEP 2 – IAM Role Setup

IAM role enabling EC2 to securely interact with ECR and ECS without access keys.

### STEP 3 – Prepare EC2 Server

System updates, Docker installation, AWS CLI v2 installation, and IAM verification.

### STEP 4 – Install and Configure Jenkins

Java installation, Jenkins service setup, plugin installation, and Docker permissions.

### STEP 5 – Connect Jenkins with GitHub

Pipeline configured using Jenkinsfile pulled directly from source control.

### STEP 6 – Build Docker Image (CI)

Docker image built and tagged using Jenkins build numbers.

### STEP 7 – Push Docker Image to Amazon ECR

Secure authentication and automated image publishing.

### STEP 8 – Create ECS Cluster

Serverless ECS cluster using AWS Fargate.

### STEP 9 – Create ECS Task Definition

Container configuration, execution role, and logging setup.

### STEP 10 – Create ECS Service

Service definition with auto-healing and desired task count.

### STEP 11 – Create Application Load Balancer

Public ALB with HTTP listener.

### STEP 12 – Create Target Group

IP-based target group with health checks.

### STEP 13 – Attach Target Group to ECS Service

Traffic routing from ALB to ECS tasks with zero downtime.

---

## 12. Monitoring & Observability

This project includes production-ready observability:

* Centralized ECS logs in CloudWatch
* ALB health and error monitoring
* ECS service stability metrics
* SNS-based alerting
* Failure simulation for validation

Detailed documentation is available at:
[Automated Monitoring & Logging for Deployment Issues](./docs/monitoring-and-alerting.md)

---

## 13. Final Takeaway

This repository demonstrates how modern cloud-native applications are:

* Built
* Deployed
* Monitored
* Operated

The architecture and workflow reflect real DevOps practices used in production environments and provide a strong foundation for professional and enterprise-grade deployments.

---



Three-Tier Application Deployment on AWS EKS

A hands-on DevOps project demonstrating how to containerize and deploy a three-tier web application consisting of a React.js frontend, Node.js backend, and MongoDB database on Amazon EKS using Docker, Amazon ECR, Kubernetes, AWS Load Balancer Controller, Terraform, and CI/CD tooling.

Project goal: Build an end-to-end containerized application platform, understand the infrastructure and deployment flow, and demonstrate practical troubleshooting across Linux, AWS, Docker, Kubernetes, networking, and CI/CD.

Architecture



High-Level Flow

Developer / GitHub
        |
        v
+---------------------+
| Application Source  |
| React / Node / Mongo |
+---------------------+
        |
        v
+---------------------+
|       Docker        |
| Build Container     |
| Images              |
+---------------------+
        |
        v
+---------------------+
|    Amazon ECR       |
| Container Registry  |
+---------------------+
        |
        v
+--------------------------------------------------+
|                 AWS VPC                          |
|                                                  |
|              +------------------+                |
|              |    Amazon EKS    |                |
|              |    Kubernetes    |                |
|              |                  |                |
|              | +--------------+ |                |
|              | | React        | |                |
|              | | Deployment   | |                |
|              | | Pods         | |                |
|              | +--------------+ |                |
|              |        |         |                |
|              |        v         |                |
|              | +--------------+ |                |
|              | | Node.js      | |                |
|              | | Deployment   | |                |
|              | | Pods         | |                |
|              | +--------------+ |                |
|              |        |         |                |
|              |        v         |                |
|              | +--------------+ |                |
|              | | MongoDB      | |                |
|              | | Deployment   | |                |
|              | | Pods         | |                |
|              | +--------------+ |                |
|              +------------------+                |
|                       |                          |
|                       v                          |
|             AWS Load Balancer Controller         |
|                       |                          |
+-----------------------|--------------------------+
                        |
                        v
                  End Users / Browser

1. Project Overview

This project demonstrates a complete application deployment workflow:

Application source code is maintained in GitHub.

Frontend, backend, and database components are packaged as Docker images.

Container images are stored in Amazon ECR.

Amazon EKS provides the managed Kubernetes environment.

Kubernetes Deployments manage application workloads.

Kubernetes Services provide stable communication endpoints for Pods.

AWS Load Balancer Controller integrates Kubernetes with AWS load-balancing resources.

End users access the application through the AWS-facing load-balancing layer.

Terraform can be used to make infrastructure provisioning repeatable.

CI/CD tooling can automate image building and deployment.

2. Application Architecture

The application is separated into three logical tiers.

Frontend Tier

Technology: React.js

Responsibilities:

Serves the user interface.

Handles browser-side application logic.

Communicates with the backend API.

Is packaged into a Docker image.

Runs as a Kubernetes workload.

Typical flow:

Browser
   |
   v
React Frontend
   |
   | HTTP/API request
   v
Node.js Backend

Backend Tier

Technology: Node.js

Responsibilities:

Exposes backend APIs.

Processes requests from the frontend.

Contains application/business logic.

Communicates with MongoDB.

Runs as a container inside Kubernetes.

Typical flow:

React Frontend
      |
      v
Node.js API
      |
      v
MongoDB

Database Tier

Technology: MongoDB

Responsibilities:

Stores application data.

Provides persistence for the backend.

Is isolated from direct end-user access.

Runs as part of the Kubernetes/containerized application in this implementation.

Database persistence, storage configuration, backup strategy, and high-availability behavior should be documented separately once implemented.

3. Technology Stack

Area

Technology

Source Control

Git, GitHub

Frontend

React.js

Backend

Node.js

Database

MongoDB

Containerization

Docker

Container Registry

Amazon ECR

Container Orchestration

Kubernetes

Managed Kubernetes

Amazon EKS

AWS Integration

AWS Load Balancer Controller

Cloud

AWS

Infrastructure as Code

Terraform

CI/CD

Jenkins / GitHub Actions

Monitoring

Prometheus, Grafana

Linux

Ubuntu

CLI Tools

AWS CLI, kubectl, eksctl, Helm

The referenced challenge repository also contains material for Jenkins pipelines, Terraform-based Jenkins infrastructure, Helm-based monitoring, Prometheus/Grafana, and ArgoCD/GitOps. Those components should only be marked as implemented after they have actually been configured and tested in this project.

4. End-to-End Deployment Flow

Step 1 — Source Code

Application source code is stored in GitHub.

GitHub Repository
       |
       +-- Frontend
       +-- Backend
       +-- Database configuration
       +-- Dockerfiles
       +-- Kubernetes manifests
       +-- Infrastructure code

Git provides version control, change tracking, collaboration, and rollback of source changes.

Step 2 — Build Docker Images

Each application component is containerized.

React Source
    |
    v
Dockerfile
    |
    v
Frontend Image

Node.js Source
    |
    v
Dockerfile
    |
    v
Backend Image

Docker provides a consistent runtime environment by packaging the application with its required dependencies.

5. Amazon ECR

Amazon Elastic Container Registry is used as the image registry.

Example:

Amazon ECR
|
+-- frontend repository
|      |
|      +-- frontend:v1
|      +-- frontend:latest
|
+-- backend repository
       |
       +-- backend:v1
       +-- backend:latest

Typical workflow:

docker build -t frontend:latest .
docker tag frontend:latest <account>.dkr.ecr.<region>.amazonaws.com/frontend:latest
docker push <account>.dkr.ecr.<region>.amazonaws.com/frontend:latest

Kubernetes deployments then reference the ECR image.

6. Amazon EKS

Amazon EKS provides the managed Kubernetes environment.

EKS Cluster
|
+-- Kubernetes Control Plane
|
+-- Worker Nodes
      |
      +-- Frontend Pods
      +-- Backend Pods
      +-- MongoDB Pods

The project uses eksctl and Kubernetes tooling for cluster setup and workload management.

Basic verification:

kubectl get nodes
kubectl get pods -A
kubectl get deployments
kubectl get services

7. Kubernetes Deployment Model

Each application tier is represented as a Kubernetes workload.

Frontend

Deployment
    |
    +-- Pod
    +-- Pod
    |
    +-- Service

Backend

Deployment
    |
    +-- Pod
    +-- Pod
    |
    +-- Service

MongoDB

Deployment
    |
    +-- Pod
    +-- Pod
    |
    +-- Service

The exact replica count and persistence configuration should match the manifests actually used in this project.

8. Kubernetes Objects Used

Deployment

A Deployment manages the desired state of application Pods.

It can be used to:

Maintain the desired replica count.

Replace failed Pods.

Perform controlled rollouts.

Maintain a consistent workload definition.

Pod

A Pod is the smallest deployable Kubernetes unit and contains one or more containers.

Useful commands:

kubectl get pods
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl exec -it <pod-name> -- /bin/sh

Service

A Kubernetes Service provides a stable network endpoint for Pods whose IP addresses can change.

Typical flow:

Frontend
   |
   v
backend-service
   |
   v
Backend Pod

Useful commands:

kubectl get svc
kubectl describe svc <service-name>
kubectl get endpoints

9. AWS Load Balancer Controller

The AWS Load Balancer Controller connects Kubernetes resources with AWS load-balancing services.

Conceptually:

Internet
   |
   v
AWS Load Balancer
   |
   v
Kubernetes Service / Ingress
   |
   v
Application Pods

The controller requires AWS IAM permissions and an IAM-enabled Kubernetes service account.

Verification:

kubectl get deployment \
  -n kube-system \
  aws-load-balancer-controller

Troubleshooting:

kubectl get pods -n kube-system
kubectl logs -n kube-system deployment/aws-load-balancer-controller

10. Networking

The project involves multiple networking layers.

Application-level networking

Browser
   |
 HTTP/HTTPS
   |
Load Balancer
   |
Kubernetes Service
   |
Pod

Kubernetes networking

Pods receive network identities that can change during their lifecycle, so Services provide stable access.

Important concepts:

Pod IP

Service IP

Port

Target Port

Node Port

Ingress / Load Balancer

DNS

11. AWS IAM

IAM controls access to AWS resources.

Conceptually:

IAM User / Role
      |
      v
Permissions Policy
      |
      v
AWS API

For production environments, avoid broad permissions and long-lived access keys wherever possible. Prefer role-based access and temporary credentials when supported.

Never commit the following to GitHub:

AWS Access Key
AWS Secret Key
Kubeconfig credentials
Private SSH keys
Database passwords
Application secrets

12. Terraform

Terraform represents infrastructure as code and makes provisioning repeatable.

Typical workflow:

terraform init
terraform fmt
terraform validate
terraform plan
terraform apply

Before destroying infrastructure:

terraform plan

To remove Terraform-managed resources:

terraform destroy

Core concepts:

Providers

Resources

Variables

Outputs

State

Plan

Apply

Drift

Recommended structure:

terraform/
├── provider.tf
├── variables.tf
├── outputs.tf
├── vpc.tf
├── iam.tf
└── eks.tf

13. CI/CD

The CI/CD objective is to automate:

Developer Push
      |
      v
Source Checkout
      |
      v
Build Application
      |
      v
Build Docker Image
      |
      v
Push Image to ECR
      |
      v
Deploy to EKS
      |
      v
Verify Deployment

Typical pipeline stages:

Checkout
   |
Build
   |
Test
   |
Docker Build
   |
ECR Push
   |
Kubernetes Deployment
   |
Health Verification

Secrets should be stored in the CI/CD platform's secret-management mechanism rather than hard-coded in pipeline files.

14. Monitoring and Logging

Monitoring should help answer:

Is the application running?

Are Pods healthy?

Are CPU/memory levels increasing?

Are containers restarting?

Are requests failing?

Did a deployment introduce failures?

Kubernetes checks:

kubectl get pods
kubectl get events
kubectl describe pod <pod>
kubectl logs <pod>
kubectl top pods
kubectl top nodes

Conceptual monitoring flow:

Application
    |
    +---- Logs
    |
    +---- Metrics
             |
             v
        Prometheus
             |
             v
          Grafana

15. Troubleshooting Approach

The project follows a layer-by-layer troubleshooting model rather than changing settings randomly.

Pod is not running

kubectl get pods
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl get events

Possible causes:

Image cannot be pulled

Application process exits

Configuration error

Missing environment variable

Resource constraints

Networking issue

ImagePullBackOff

Check:

kubectl describe pod <pod-name>

Verify:

ECR repository

Image name

Image tag

Registry authentication/permissions

Node access to ECR

CrashLoopBackOff

Check:

kubectl logs <pod-name>
kubectl logs <pod-name> --previous
kubectl describe pod <pod-name>

Investigate:

Application startup failure

Environment configuration

Dependency connectivity

Port configuration

Resource problems

Load balancer or ingress issue

Check from Kubernetes outward:

kubectl get ingress
kubectl describe ingress <ingress-name>
kubectl get svc
kubectl get endpoints
kubectl get pods

Then inspect AWS-side resources:

Load Balancer
    |
Target Groups
    |
Target Health
    |
Security Groups
    |
Network connectivity

Troubleshoot one layer at a time:

User
 ↓
Load Balancer
 ↓
Kubernetes Service
 ↓
Endpoints
 ↓
Pod
 ↓
Container
 ↓
Application

16. Linux Administration

Linux is used for server administration and troubleshooting.

Common commands used during the project:

pwd
ls
cd
mkdir
cp
mv
rm
cat
less
grep
find
tail
head

Permissions:

ls -l
chmod
chown

Processes:

ps
top
kill

System resources:

df -h
du -sh
free -m
uptime

Networking:

ip addr
ip route
ss -tulpn
curl
ping
nslookup

Logs:

journalctl
tail -f

17. Security Considerations

Security practices considered in this project include:

Avoid committing secrets to Git.

Prefer IAM roles and temporary credentials where possible.

Restrict network access using security groups.

Keep databases inaccessible directly from the public internet.

Use private container registries such as ECR.

Store CI/CD credentials as secrets.

Minimize IAM privileges according to the required task.

For a production implementation, additional controls could include:

AWS Secrets Manager / Parameter Store

Encryption at rest

Encryption in transit

Kubernetes RBAC

Pod security controls

Image vulnerability scanning

Centralized audit logging

Backup and disaster recovery

18. Deployment Commands

Example EKS workflow:

aws configure

eksctl create cluster \
  --name three-tier-cluster \
  --region <region> \
  --node-type <node-type> \
  --nodes-min 2 \
  --nodes-max 2

Configure kubectl:

aws eks update-kubeconfig \
  --region <region> \
  --name three-tier-cluster

Verify:

kubectl get nodes

Deploy manifests:

kubectl apply -f kubernetes/

Verify:

kubectl get pods -A
kubectl get svc -A
kubectl get deployments -A

Cleanup:

kubectl delete -f kubernetes/
eksctl delete cluster \
  --name three-tier-cluster \
  --region <region>

Always verify the AWS account, region, cluster name, namespaces, load balancers, ECR repositories, and other billable resources before cleanup.

19. Validation Checklist

AWS

Correct AWS account and region selected

IAM permissions verified

EKS cluster created

Worker nodes registered

Networking verified

Docker

Frontend image builds successfully

Backend image builds successfully

Images run locally

Image tags are consistent

ECR

Repositories created

Images pushed successfully

Kubernetes references correct ECR image URIs

Kubernetes

Namespace created

Deployments created

Pods running

Services created

Logs checked

No unexpected restart loops

Load Balancer

AWS Load Balancer Controller healthy

Load balancer created

Targets healthy

Application accessible externally

CI/CD

Source checkout works

Docker build works

Image push works

Deployment works

Deployment can be verified

Monitoring

Logs can be retrieved

Metrics can be inspected

Pod and node health can be checked

20. Interview Knowledge Checklist

This project is intended to support practical discussion of:

Linux

Process management

File permissions

Services

Logs

CPU and memory troubleshooting

Disk troubleshooting

Networking commands

AWS

VPC

Subnets

IAM

Security Groups

ECR

EKS

Load balancing

AWS networking fundamentals

Docker

Images vs containers

Dockerfile

Layers

Ports

Networks

Volumes

Container logs

Image tagging

Kubernetes

Pods

Deployments

Services

Namespaces

Labels and selectors

Replica management

Logs and events

Troubleshooting

Ingress/load-balancing concepts

Terraform

Infrastructure as Code

Providers

Resources

Variables

State

Plan

Apply

Drift

CI/CD

Continuous Integration

Continuous Delivery/Deployment

Pipeline stages

Artifact/container image flow

Secret management

Deployment verification

Monitoring

Metrics

Logs

Prometheus

Grafana

Application health

Resource utilization

21. Troubleshooting Scenarios Practiced

The project should be used to practice deliberately breaking and fixing common failures:

Pod remains pending.

Image cannot be pulled from ECR.

Container exits immediately.

Pod enters CrashLoopBackOff.

Service has no healthy endpoints.

Application is reachable inside the cluster but not externally.

Load balancer target is unhealthy.

DNS or routing prevents application access.

Container port and Service target port do not match.

Deployment succeeds but the new application version is not serving.

The goal is to follow a repeatable diagnostic process rather than memorize isolated fixes.

22. Learning Outcomes

By completing this project, the target end-to-end workflow is:

Source Code
    ↓
Git
    ↓
Docker Build
    ↓
Amazon ECR
    ↓
Amazon EKS
    ↓
Kubernetes Deployment
    ↓
Kubernetes Service
    ↓
AWS Load Balancer
    ↓
End User

Operational workflow:

Monitor
   ↓
Detect Problem
   ↓
Collect Logs / Metrics
   ↓
Identify Failure Layer
   ↓
Apply Fix
   ↓
Verify Health

23. Project Status

Update this section as implementation progresses.

[ ] Application source code configured
[ ] Dockerfiles created
[ ] Docker images built
[ ] ECR repositories created
[ ] Images pushed to ECR
[ ] EKS cluster created
[ ] Kubernetes manifests deployed
[ ] Frontend verified
[ ] Backend verified
[ ] MongoDB verified
[ ] AWS Load Balancer Controller configured
[ ] Application exposed externally
[ ] Terraform infrastructure added
[ ] CI/CD pipeline added
[ ] Monitoring added
[ ] Troubleshooting scenarios tested
[ ] Architecture documentation completed

24. References

This project uses the following public project as a learning reference:

Original reference repository:
https://github.com/LondheShubham153/TWSThreeTierAppChallenge

The reference repository contains material covering the application challenge, AWS EKS, Kubernetes manifests, Jenkins, Terraform, monitoring, and GitOps.

This repository documents the implementation, configuration, troubleshooting, and learning outcomes performed here and does not represent the referenced project as original work.

25. Author

Aditya Gund

Junior DevOps / Cloud Engineering Project

Focus Areas:

AWS
Linux
Docker
Kubernetes
Terraform
CI/CD
Networking
Monitoring
Automation

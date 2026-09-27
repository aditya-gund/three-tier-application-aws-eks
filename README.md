<div align="center">

Three-Tier Application Deployment on AWS EKS

React.js • Node.js • MongoDB • Docker • Amazon ECR • Kubernetes • Amazon EKS

A hands-on DevOps project demonstrating the containerization, deployment, exposure, and troubleshooting of a three-tier application on AWS using Kubernetes and Amazon EKS.

<br/>


<img width="1774" height="887" alt="Achitecture" src="https://github.com/user-attachments/assets/79012d08-c3e3-4faf-8807-1de9526227ce" />

</div>

📌 Project Overview

This project deploys a three-tier web application on Amazon EKS:

Tier

Technology

Responsibility

🎨 Presentation

React.js

User interface and browser-side application

⚙️ Application

Node.js

Backend APIs and application/business logic

🗄️ Data

MongoDB

Application data persistence

The application components are containerized with Docker, stored in Amazon ECR, and deployed as Kubernetes workloads on Amazon EKS.

External access is provided through the AWS load-balancing layer integrated with Kubernetes through the AWS Load Balancer Controller.

Project objective: understand the complete DevOps flow from source code → container image → registry → Kubernetes deployment → AWS load balancer → end user, while developing practical troubleshooting skills.

🏗️ Architecture

<p align="center">
  <img src="docs/architecture.png" alt="Three-Tier Application AWS EKS Architecture" width="100%">
</p>

End-to-End Flow

Developer
    │
    ▼
GitHub Repository
    │
    ├───────────────┐
    │               │
    ▼               ▼
React.js         Node.js
    │               │
    └───────┬───────┘
            │
            ▼
          Docker
            │
            ▼
       Amazon ECR
            │
            ▼
     Amazon EKS Cluster
            │
     ┌──────┼───────────┐
     │      │           │
     ▼      ▼           ▼
 Frontend Backend     MongoDB
   Pods     Pods        Pods
     │      │           │
     └──────┴───────────┘
            │
            ▼
AWS Load Balancer Controller
            │
            ▼
      AWS Load Balancer
            │
            ▼
       End User / Browser

🔄 Deployment Lifecycle

flowchart LR
    A[Source Code] --> B[GitHub]
    B --> C[Docker Build]
    C --> D[Amazon ECR]
    D --> E[Amazon EKS]
    E --> F[Kubernetes Deployments]
    F --> G[Kubernetes Services]
    G --> H[AWS Load Balancer]
    H --> I[End User]

What happens at each stage?

1. Source Code

Application code is maintained in Git and GitHub.

2. Containerization

Docker packages each application component with its runtime dependencies.

3. Image Registry

Container images are tagged and pushed to Amazon ECR.

4. Kubernetes Deployment

Amazon EKS runs the containerized workloads through Kubernetes Deployments and Pods.

5. Service Discovery

Kubernetes Services provide stable endpoints for application communication.

6. External Access

The AWS Load Balancer Controller integrates Kubernetes with AWS load-balancing resources.

7. Application Access

End users access the application through the externally exposed endpoint.

🧩 Application Components

Frontend — React.js

The frontend is responsible for:

Rendering the application UI.

Handling browser-side logic.

Making API requests to the backend.

Running as a containerized workload.

Being exposed through the application entry point.

Typical flow:

Browser
   │
   ▼
React.js
   │
   │ API Request
   ▼
Node.js Backend

Backend — Node.js

The backend is responsible for:

Receiving API requests.

Processing application logic.

Communicating with MongoDB.

Returning responses to the React frontend.

Running as a Kubernetes workload.

Typical flow:

React.js
   │
   ▼
Node.js API
   │
   ▼
MongoDB

Database — MongoDB

MongoDB provides the application's data layer.

Responsibilities:

Store application data.

Serve database requests from the backend.

Remain inaccessible directly from public users.

Run within the application's containerized/Kubernetes environment for this implementation.

Database persistence and production-grade storage behavior should be validated separately against the final Kubernetes manifests before treating them as production-ready.

🐳 Containerization

Each major application component is packaged as a Docker image.

React Source
    │
    ▼
Dockerfile
    │
    ▼
Frontend Image
    │
    ▼
Amazon ECR

Node.js Source
    │
    ▼
Dockerfile
    │
    ▼
Backend Image
    │
    ▼
Amazon ECR

Containerization provides:

Consistent application environments.

Repeatable deployments.

Dependency isolation.

Portable workloads.

Immutable deployment artifacts.

Useful Docker commands

docker build -t frontend:v1 .
docker images

docker run -d -p 8080:80 frontend:v1

docker ps
docker logs <container>
docker inspect <container>

docker stop <container>
docker rm <container>

📦 Amazon ECR

Amazon Elastic Container Registry stores the Docker images consumed by the EKS workloads.

Example repository model:

Amazon ECR
│
├── frontend
│   ├── v1
│   └── latest
│
├── backend
│   ├── v1
│   └── latest
│
└── mongodb
    └── ...

Example authentication and push workflow:

aws ecr get-login-password --region <region> \
  | docker login \
  --username AWS \
  --password-stdin \
  <account-id>.dkr.ecr.<region>.amazonaws.com

docker build -t frontend:v1 ./frontend

docker tag frontend:v1 \
  <account-id>.dkr.ecr.<region>.amazonaws.com/frontend:v1

docker push \
  <account-id>.dkr.ecr.<region>.amazonaws.com/frontend:v1

☸️ Amazon EKS

Amazon EKS provides the managed Kubernetes platform used to run the application.

Logical cluster model

Amazon EKS
│
├── Control Plane
│
└── Worker Nodes
    │
    ├── Frontend Pods
    ├── Backend Pods
    └── MongoDB Pods

The cluster can be created and managed using:

AWS CLI

eksctl

kubectl

Cluster configuration

eksctl create cluster \
  --name three-tier-cluster \
  --region <region> \
  --node-type <node-type> \
  --nodes-min 2 \
  --nodes-max 2

Configure local Kubernetes access:

aws eks update-kubeconfig \
  --region <region> \
  --name three-tier-cluster

Verify:

kubectl get nodes
kubectl get pods -A

☸️ Kubernetes Architecture

The application is represented through Kubernetes resources.

                  EKS Cluster
                      │
        ┌─────────────┼─────────────┐
        │             │             │
        ▼             ▼             ▼
   Deployment    Deployment    Deployment
    Frontend      Backend        MongoDB
        │             │             │
      Pods          Pods           Pods
        │             │             │
        ▼             ▼             ▼
    Frontend      Backend        MongoDB
     Service       Service        Service

Deployment

A Kubernetes Deployment describes the desired state of an application workload.

Example concepts:

replicas: 2
selector:
  matchLabels:
    app: backend

A Deployment can:

Maintain the desired number of Pods.

Replace failed Pods.

Perform rolling updates.

Keep workload configuration declarative.

Useful commands:

kubectl get deployments
kubectl describe deployment <deployment-name>
kubectl rollout status deployment/<deployment-name>
kubectl rollout history deployment/<deployment-name>

Pods

Pods are the smallest deployable units in Kubernetes.

Useful commands:

kubectl get pods
kubectl get pods -o wide
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl logs <pod-name> --previous

For interactive troubleshooting:

kubectl exec -it <pod-name> -- /bin/sh

Services

A Service provides a stable network abstraction for Pods.

Frontend
   │
   ▼
backend-service
   │
   ▼
Backend Pods

Useful commands:

kubectl get svc
kubectl describe svc <service-name>
kubectl get endpoints

🌐 AWS Load Balancer Controller

The AWS Load Balancer Controller enables Kubernetes resources to provision and manage AWS load-balancing resources.

Logical flow:

Internet
   │
   ▼
AWS Load Balancer
   │
   ▼
Kubernetes Ingress / Service
   │
   ▼
Application Pods

Controller verification:

kubectl get deployment \
  -n kube-system \
  aws-load-balancer-controller

Controller logs:

kubectl logs \
  -n kube-system \
  deployment/aws-load-balancer-controller

🌐 Networking

Understanding the networking path is a major part of this project.

User Browser
     │
     ▼
AWS Load Balancer
     │
     ▼
Kubernetes Service
     │
     ▼
Backend / Frontend Pod
     │
     ▼
Container Port

Important concepts:

IP address

Subnet

Route table

Security Group

DNS

TCP/IP

HTTP/HTTPS

Kubernetes Pod IP

Kubernetes Service IP

Port

Target Port

Load balancing

Ingress

Basic Linux networking commands

ip addr
ip route
ss -tulpn
ping <host>
curl <url>
nslookup <domain>

🏗️ Infrastructure as Code — Terraform

Terraform is used to model infrastructure declaratively.

Typical workflow:

terraform init
terraform fmt
terraform validate
terraform plan
terraform apply

Inspect state:

terraform state list
terraform show

Remove managed infrastructure when appropriate:

terraform destroy

Core Terraform concepts

Concept

Purpose

Provider

Connects Terraform to an infrastructure platform

Resource

Defines infrastructure to manage

Variable

Makes configurations reusable

Output

Exposes useful resource information

State

Tracks Terraform-managed infrastructure

Plan

Previews changes

Apply

Executes changes

Example structure:

terraform/
├── provider.tf
├── variables.tf
├── outputs.tf
├── vpc.tf
├── iam.tf
└── eks.tf

🔁 CI/CD

The target delivery workflow is:

Developer Push
      │
      ▼
Source Checkout
      │
      ▼
Build / Test
      │
      ▼
Docker Build
      │
      ▼
Push Image to ECR
      │
      ▼
Deploy to EKS
      │
      ▼
Verify Application

Typical pipeline stages:

Checkout
   ↓
Build
   ↓
Test
   ↓
Docker Build
   ↓
ECR Push
   ↓
Kubernetes Deployment
   ↓
Health Check

CI/CD principles demonstrated

Automated repeatable deployments

Immutable container artifacts

Secret management

Deployment verification

Failure visibility

Rollback awareness

📊 Monitoring & Logging

Monitoring answers:

Is the application healthy?

Are Pods restarting?

Is CPU/memory usage increasing?

Are requests failing?

Is the deployment stable?

The monitoring stack can be represented as:

Application / Kubernetes
         │
         ├── Logs
         │
         └── Metrics
               │
               ▼
           Prometheus
               │
               ▼
            Grafana

Useful Kubernetes diagnostics:

kubectl get pods
kubectl get events
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl top pods
kubectl top nodes

🔧 Troubleshooting Playbook

A key objective of this project is learning to troubleshoot systematically rather than changing random configuration.

1. Pod is not starting

kubectl get pods
kubectl describe pod <pod-name>
kubectl get events

Check for:

Image issues

Scheduling problems

Missing configuration

Resource constraints

Volume problems

2. ImagePullBackOff

kubectl describe pod <pod-name>

Validate:

ECR repository exists.

Image tag is correct.

Image URL is correct.

Node/workload has required permissions.

Image is available in the correct AWS region/account.

3. CrashLoopBackOff

kubectl logs <pod-name>
kubectl logs <pod-name> --previous
kubectl describe pod <pod-name>

Investigate:

Application startup errors.

Incorrect environment variables.

Missing dependencies.

Port configuration.

Database connectivity.

Runtime exceptions.

4. Service exists but application is unreachable

Check layer by layer:

kubectl get svc
kubectl describe svc <service-name>
kubectl get endpoints
kubectl get pods -o wide

Then validate:

Load Balancer
    ↓
Service
    ↓
Endpoints
    ↓
Pod
    ↓
Container
    ↓
Application

5. Load Balancer target is unhealthy

Check:

kubectl describe ingress <ingress-name>
kubectl get svc
kubectl get endpoints

Then inspect the AWS load balancer and target health.

Potential causes:

Wrong Service port.

Wrong target port.

Pod not ready.

Application not listening on expected port.

Security Group/networking issue.

Incorrect health-check configuration.

🐧 Linux Administration

Linux is the operational foundation for the deployment environment.

Files & directories

pwd
ls -la
cd
mkdir
touch
cp
mv
rm
find
grep

Permissions

ls -l
chmod
chown

Processes

ps
ps aux
top
kill

Resources

df -h
du -sh
free -m
uptime

Services & logs

systemctl status <service>
journalctl -u <service>
tail -f <log-file>

Networking

ip addr
ip route
ss -tulpn
curl
ping
nslookup

🔐 Security

Security practices for this project include:

Do not commit AWS access keys.

Do not commit database passwords.

Do not commit private SSH keys.

Store CI/CD secrets securely.

Limit AWS IAM permissions.

Restrict network traffic with security groups.

Keep the database tier away from direct public access.

Use private container registries where appropriate.

Use HTTPS for production traffic.

Apply Kubernetes RBAC where applicable.

Never commit

.env
*.pem
*.key
AWS access keys
AWS secret keys
kubeconfig files
database passwords
CI/CD credentials

📁 Suggested Repository Structure

three-tier-application-aws-eks/
│
├── README.md
│
├── docs/
│   └── architecture.png
│
├── frontend/
│   ├── Dockerfile
│   └── ...
│
├── backend/
│   ├── Dockerfile
│   └── ...
│
├── kubernetes/
│   ├── namespace.yaml
│   ├── frontend-deployment.yaml
│   ├── frontend-service.yaml
│   ├── backend-deployment.yaml
│   ├── backend-service.yaml
│   ├── mongodb-deployment.yaml
│   └── ingress.yaml
│
├── terraform/
│   ├── provider.tf
│   ├── variables.tf
│   ├── outputs.tf
│   ├── vpc.tf
│   ├── iam.tf
│   └── eks.tf
│
├── scripts/
│   ├── deploy.sh
│   └── health-check.sh
│
└── .gitignore

🚀 Quick Deployment Workflow

1. Configure AWS

aws configure
aws sts get-caller-identity

2. Create EKS

eksctl create cluster \
  --name three-tier-cluster \
  --region <region> \
  --node-type <node-type> \
  --nodes-min 2 \
  --nodes-max 2

3. Configure Kubernetes

aws eks update-kubeconfig \
  --region <region> \
  --name three-tier-cluster

4. Verify the cluster

kubectl get nodes
kubectl get pods -A

5. Deploy workloads

kubectl apply -f kubernetes/

6. Verify deployments

kubectl get deployments
kubectl get pods
kubectl get svc

7. Check application logs

kubectl logs <pod-name>

8. Check external access

kubectl get ingress
kubectl get svc

🧪 Validation Checklist

Infrastructure

AWS account and region verified

IAM permissions verified

EKS cluster available

Worker nodes registered

Kubernetes context configured

Application

Frontend builds successfully

Backend builds successfully

MongoDB configuration validated

Docker images created

Images pushed to ECR

Kubernetes

Namespace created

Deployments applied

Pods running

Services created

Endpoints populated

Logs checked

No unexpected restarts

AWS Integration

AWS Load Balancer Controller installed

Required IAM integration configured

Load balancer provisioned

Targets healthy

Application reachable externally

Operations

Terraform validated

CI/CD workflow tested

Monitoring checked

Failure scenarios tested

Cleanup procedure verified

🎯 Interview Preparation

This project should be explainable at four levels:

Level 1 — Architecture

Be able to draw:

User
 ↓
AWS Load Balancer
 ↓
Kubernetes Service
 ↓
Pod
 ↓
Container
 ↓
Application

Level 2 — Kubernetes

Be able to explain:

Pod

Deployment

Replica

Service

Namespace

Ingress

Labels/selectors

Rolling updates

Level 3 — AWS

Be able to explain:

EKS

ECR

IAM

VPC

Subnets

Security Groups

Load balancing

Level 4 — Troubleshooting

Be able to reason through:

"Pod is CrashLoopBackOff."

        ↓
kubectl get pods
        ↓
kubectl describe pod
        ↓
kubectl logs
        ↓
Check configuration
        ↓
Check dependencies
        ↓
Fix
        ↓
Verify

💡 Key Engineering Takeaways

The project demonstrates the relationship between:

Application Development
        ↓
Containerization
        ↓
Image Management
        ↓
Orchestration
        ↓
Networking
        ↓
Cloud Infrastructure
        ↓
Automation
        ↓
Monitoring
        ↓
Troubleshooting

The important DevOps skill is not simply knowing individual commands. It is understanding how these layers interact and how to identify the failing layer when something breaks.

📌 Project Status

Update this section as each component is actually implemented and tested.

Component

Status

React.js application

⬜

Node.js backend

⬜

MongoDB

⬜

Docker images

⬜

Amazon ECR

⬜

Amazon EKS

⬜

Kubernetes manifests

⬜

AWS Load Balancer Controller

⬜

External application access

⬜

Terraform

⬜

CI/CD

⬜

Prometheus

⬜

Grafana

⬜

Troubleshooting scenarios

⬜

<div align="center">

Built as a p<img width="1774" height="887" alt="Achitecture" src="https://github.com/user-attachments/assets/931589cd-caf8-4d02-9d7e-8db5a6962775" />

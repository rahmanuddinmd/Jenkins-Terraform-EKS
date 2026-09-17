# Project Index

## Jenkins + Terraform + Amazon EKS

This index provides a quick map of the project, its files, technologies,
workflow, and operational steps.

------------------------------------------------------------------------

## 1. Project Purpose

Automate Amazon EKS infrastructure provisioning using:

``` text
GitHub
  ↓
Jenkins
  ↓
Terraform
  ↓
AWS VPC
  ↓
Amazon EKS
  ↓
Kubernetes
```

The pipeline supports:

-   Infrastructure creation with `apply`
-   Infrastructure deletion with `destroy`
-   Terraform validation
-   Terraform planning
-   Manual approval before infrastructure changes

------------------------------------------------------------------------

## 2. Technology Stack

  Layer                     Technology   Role
  ------------------------- ------------ ---------------------------------------
  Source Control            GitHub       Store Terraform and project files
  Automation                Jenkins      Execute infrastructure pipeline
  IaC                       Terraform    Define AWS infrastructure
  Cloud                     AWS          Host infrastructure
  Container Orchestration   Amazon EKS   Managed Kubernetes
  Networking                Amazon VPC   EKS network
  Compute                   EC2          Jenkins and Kubernetes worker nodes
  Kubernetes CLI            kubectl      Manage the EKS cluster
  Cloud CLI                 AWS CLI      Authenticate and configure EKS access

------------------------------------------------------------------------

## 3. Repository Files

``` text
Jenkins-Terraform-EKS-main/
│
├── README.md
├── INDEX.md
├── Creation of EKS Cluster.txt
│
└── terraform/
    ├── provider.tf
    ├── vpc.tf
    └── eks.tf
```

### File Responsibilities

  -----------------------------------------------------------------------
  File                                Purpose
  ----------------------------------- -----------------------------------
  `README.md`                         Complete project documentation

  `INDEX.md`                          Quick navigation/reference

  `Creation of EKS Cluster.txt`       Original installation and execution
                                      notes

  `terraform/provider.tf`             AWS provider and project
                                      variables/local values

  `terraform/vpc.tf`                  VPC and subnet infrastructure

  `terraform/eks.tf`                  EKS cluster and managed node group
  -----------------------------------------------------------------------

------------------------------------------------------------------------

## 4. AWS Configuration

### Region

``` text
us-east-1
```

### Cluster

``` text
kastro-eks-cluster
```

### VPC

``` text
10.123.0.0/16
```

### Availability Zones

``` text
us-east-1a
us-east-1b
```

### Public Subnets

``` text
10.123.1.0/24
10.123.2.0/24
```

### Private Subnets

``` text
10.123.3.0/24
10.123.4.0/24
```

### Intra Subnets

``` text
10.123.5.0/24
10.123.6.0/24
```

------------------------------------------------------------------------

## 5. EKS Configuration

### Add-ons

``` text
CoreDNS
kube-proxy
VPC CNI
```

### Managed Node Group

``` text
Name: cluster-wg
Minimum: 1
Desired: 1
Maximum: 2
Instance Type: t3.large
Capacity: SPOT
```

------------------------------------------------------------------------

## 6. Main Workflow

``` text
1. Prepare AWS
       ↓
2. Prepare Jenkins
       ↓
3. Configure Jenkins credentials
       ↓
4. Clone GitHub repository
       ↓
5. Terraform init
       ↓
6. Terraform validate
       ↓
7. Terraform plan
       ↓
8. Manual approval
       ↓
9. Terraform apply
       ↓
10. EKS cluster created
       ↓
11. Configure kubectl
       ↓
12. Deploy/test Kubernetes workload
```

------------------------------------------------------------------------

## 7. Cleanup Workflow

``` text
Jenkins
   ↓
Build with Parameters
   ↓
action = destroy
   ↓
Terraform destroy
   ↓
Verify AWS cleanup
   ↓
Terminate Jenkins EC2 if no longer required
```

------------------------------------------------------------------------

## 8. Key Terraform Commands

``` bash
cd terraform

terraform init
terraform fmt
terraform validate
terraform plan
terraform apply
terraform destroy
terraform state list
```

------------------------------------------------------------------------

## 9. Key AWS Commands

``` bash
aws --version
aws sts get-caller-identity
aws eks list-clusters --region us-east-1
```

Configure Kubernetes access:

``` bash
aws eks update-kubeconfig \
  --region us-east-1 \
  --name kastro-eks-cluster
```

------------------------------------------------------------------------

## 10. Key Kubernetes Commands

``` bash
kubectl config current-context
kubectl get nodes
kubectl get pods -A

kubectl run nginx --image=nginx

kubectl get pods
kubectl describe pod nginx
kubectl logs nginx

kubectl delete pod nginx
```

------------------------------------------------------------------------

## 11. Jenkins Pipeline Stages

``` text
Clone the Code
      ↓
Terraform Initialization
      ↓
Terraform Validation
      ↓
Infrastructure Checks
      ↓
Manual Approval
      ↓
Create/Destroy EKS Cluster
```

------------------------------------------------------------------------

## 12. Jenkins Parameter

Create a Choice Parameter:

``` text
Name:
action
```

Values:

``` text
apply
destroy
```

Use:

``` text
Build with Parameters
```

to select the operation.

------------------------------------------------------------------------

## 13. Security Checklist

Before using this project beyond a learning environment:

-   [ ] Do not commit AWS access keys.
-   [ ] Use IAM roles where possible.
-   [ ] Apply least-privilege permissions.
-   [ ] Secure Jenkins.
-   [ ] Restrict EKS API access appropriately.
-   [ ] Protect Terraform state.
-   [ ] Use remote Terraform state for team environments.
-   [ ] Add infrastructure security scanning.
-   [ ] Review Spot instance suitability.
-   [ ] Review all AWS resources after destroying the cluster.

------------------------------------------------------------------------

## 14. Recommended Improvements

Future versions can add:

``` text
Jenkinsfile
variables.tf
outputs.tf
versions.tf
remote Terraform state
S3 backend
state locking
Terraform security scanning
Trivy
Checkov
tflint
Prometheus
Grafana
CloudWatch
Slack/Teams notifications
Dev/QA/Prod environments
```

------------------------------------------------------------------------

## 15. Learning Areas

This project covers:

### AWS

``` text
IAM
VPC
Subnets
NAT Gateway
EC2
EKS
```

### Terraform

``` text
Provider
Modules
State
Init
Validate
Plan
Apply
Destroy
```

### Jenkins

``` text
Pipeline
Credentials
Parameters
Stages
Manual approval
Automation
```

### Kubernetes

``` text
Cluster
Node
Pod
kubectl
Context
Workload
```

------------------------------------------------------------------------

## 16. Project Outcome

The final result is an automated infrastructure workflow:

``` text
Developer
    |
    v
GitHub
    |
    v
Jenkins
    |
    v
Terraform
    |
    +-------------------+
    |                   |
    v                   v
AWS VPC              Amazon EKS
                        |
                        v
                  Worker Nodes
                        |
                        v
                  Kubernetes Pods
```

The project demonstrates how Infrastructure as Code and CI/CD automation
can be combined to create and manage a Kubernetes platform consistently.

------------------------------------------------------------------------

## 17. Quick Start

### Step 1

Prepare an AWS account and appropriate IAM permissions.

### Step 2

Prepare an Ubuntu Jenkins server.

### Step 3

Install:

``` text
Java
Jenkins
Terraform
AWS CLI
kubectl
```

### Step 4

Configure AWS credentials securely in Jenkins.

### Step 5

Configure the Jenkins pipeline.

### Step 6

Create the parameter:

``` text
action = apply
action = destroy
```

### Step 7

Run:

``` text
Build with Parameters → apply
```

### Step 8

Approve the Terraform plan.

### Step 9

Verify EKS in AWS.

### Step 10

Configure kubectl and test:

``` bash
kubectl get nodes
kubectl run nginx --image=nginx
kubectl get pods
```

### Step 11

When finished:

``` text
Build with Parameters → destroy
```

------------------------------------------------------------------------

## 18. Documentation Map

  Need                           Read
  ------------------------------ -------------------------------
  Complete project explanation   `README.md`
  Quick navigation               `INDEX.md`
  Original setup instructions    `Creation of EKS Cluster.txt`
  AWS/provider configuration     `terraform/provider.tf`
  VPC/networking                 `terraform/vpc.tf`
  EKS/worker nodes               `terraform/eks.tf`

------------------------------------------------------------------------

## 19. Final Architecture

``` text
                     +----------------+
                     |    GitHub      |
                     | Source Control |
                     +-------+--------+
                             |
                             v
                     +----------------+
                     |    Jenkins     |
                     | CI/CD Pipeline |
                     +-------+--------+
                             |
                             v
                     +----------------+
                     |   Terraform    |
                     | Infrastructure |
                     |   as Code      |
                     +-------+--------+
                             |
                             v
              +-----------------------------+
              |             AWS             |
              |                             |
              |  +-----------------------+  |
              |  |          VPC          |  |
              |  |                       |  |
              |  | Public / Private /    |  |
              |  | Intra Subnets         |  |
              |  |                       |  |
              |  |      NAT Gateway      |  |
              |  +-----------+-----------+  |
              |              |              |
              |              v              |
              |      +---------------+      |
              |      |     EKS       |      |
              |      |  Control Plane|      |
              |      +-------+-------+      |
              |              |              |
              |              v              |
              |      +---------------+      |
              |      | Managed Nodes |      |
              |      +-------+-------+      |
              |              |              |
              |              v              |
              |         Kubernetes         |
              +-----------------------------+
```

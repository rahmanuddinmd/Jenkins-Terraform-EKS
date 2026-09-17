# Jenkins + Terraform + Amazon EKS Automation

Automate the provisioning and destruction of an Amazon Elastic
Kubernetes Service (EKS) cluster using **Jenkins**, **Terraform**,
**AWS**, and **GitHub**.

This project demonstrates a practical DevOps Infrastructure as Code
workflow:

``` text
Developer
   |
   v
GitHub Repository
   |
   v
Jenkins Pipeline
   |
   +--> Terraform Init
   |
   +--> Terraform Validate
   |
   +--> Terraform Plan
   |
   +--> Manual Approval
   |
   +--> Terraform Apply / Destroy
   |
   v
AWS
   |
   +--> VPC
   +--> Public / Private / Intra Subnets
   +--> NAT Gateway
   +--> EKS Control Plane
   +--> EKS Managed Node Group
   +--> EKS Add-ons
```

------------------------------------------------------------------------

## Table of Contents

1.  [Project Overview](#project-overview)
2.  [Project Objectives](#project-objectives)
3.  [What This Project Uses](#what-this-project-uses)
4.  [Repository Structure](#repository-structure)
5.  [Architecture](#architecture)
6.  [How the Project Works](#how-the-project-works)
7.  [Prerequisites](#prerequisites)
8.  [AWS Requirements](#aws-requirements)
9.  [Jenkins Setup](#jenkins-setup)
10. [Terraform Configuration](#terraform-configuration)
11. [Jenkins Pipeline](#jenkins-pipeline)
12. [Running the Project](#running-the-project)
13. [Verify the EKS Cluster](#verify-the-eks-cluster)
14. [Connect to EKS with kubectl](#connect-to-eks-with-kubectl)
15. [Test Kubernetes](#test-kubernetes)
16. [Destroy the Infrastructure](#destroy-the-infrastructure)
17. [Security Recommendations](#security-recommendations)
18. [Cost Considerations](#cost-considerations)
19. [Troubleshooting](#troubleshooting)
20. [Useful Commands](#useful-commands)
21. [Production Improvements](#production-improvements)
22. [Learning Outcomes](#learning-outcomes)
23. [Future Enhancements](#future-enhancements)
24. [Conclusion](#conclusion)

------------------------------------------------------------------------

## Project Overview

The project provisions an Amazon EKS cluster through a Jenkins pipeline
instead of requiring an engineer to execute Terraform commands manually.

Terraform defines the AWS infrastructure as code. Jenkins acts as the
automation/orchestration layer. GitHub stores the source code and
provides version control.

The pipeline supports two operations:

-   `apply` - create or update the infrastructure
-   `destroy` - remove the infrastructure created by Terraform

A manual approval is included after `terraform plan` so that the
proposed infrastructure changes can be reviewed before execution.

------------------------------------------------------------------------

## Project Objectives

The main goals are:

-   Automate EKS cluster creation.
-   Use Terraform for Infrastructure as Code.
-   Use Jenkins for CI/CD-style infrastructure automation.
-   Store infrastructure code in GitHub.
-   Create a dedicated AWS VPC for EKS.
-   Create public, private, and intra subnets.
-   Configure a NAT Gateway.
-   Create an EKS control plane.
-   Create an EKS managed node group.
-   Enable common EKS add-ons.
-   Allow controlled `apply` and `destroy` operations.
-   Demonstrate Kubernetes access using `kubectl`.
-   Provide a repeatable DevOps workflow.

------------------------------------------------------------------------

## What This Project Uses

  Technology                 Purpose
  -------------------------- ------------------------------------------
  AWS                        Cloud infrastructure provider
  Amazon EKS                 Managed Kubernetes control plane
  Amazon VPC                 Network infrastructure
  EC2                        Jenkins host and EKS worker nodes
  Terraform                  Infrastructure as Code
  Jenkins                    Pipeline automation
  GitHub                     Source-code/version-control repository
  kubectl                    Kubernetes command-line client
  AWS CLI                    AWS authentication and EKS configuration
  Terraform AWS VPC Module   VPC/subnet/NAT provisioning
  Terraform AWS EKS Module   EKS cluster provisioning

### Terraform modules used

This repository uses:

``` text
terraform-aws-modules/vpc/aws
terraform-aws-modules/eks/aws
```

The EKS module is pinned to:

``` text
19.15.1
```

The VPC module uses:

``` text
~> 4.0
```

------------------------------------------------------------------------

## Repository Structure

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

### `README.md`

Main project documentation.

### `INDEX.md`

Quick navigation and project reference.

### `Creation of EKS Cluster.txt`

Step-by-step installation and Jenkins configuration notes.

### `terraform/provider.tf`

Contains:

-   AWS provider
-   AWS region
-   cluster name
-   VPC CIDR
-   Availability Zones
-   subnet CIDRs
-   common tags

### `terraform/vpc.tf`

Creates the VPC networking layer.

### `terraform/eks.tf`

Creates the EKS cluster, add-ons, and managed node group.

------------------------------------------------------------------------

# Architecture

## High-Level Architecture

``` text
                         GitHub
                           |
                           v
                    +-------------+
                    |   Jenkins   |
                    |   Pipeline  |
                    +-------------+
                           |
                           | Terraform
                           v
                    +-------------+
                    |    AWS      |
                    +-------------+
                           |
                           v
                 +-------------------+
                 |       VPC         |
                 | 10.123.0.0/16     |
                 +-------------------+
                   /       |        \
                  /        |         \
                 v         v          v
          Public Subnets Private    Intra
                         Subnets    Subnets
                             |
                             v
                       NAT Gateway
                             |
                             v
                     Amazon EKS
                   +----------------+
                   | Control Plane  |
                   +----------------+
                           |
                           v
                  Managed Node Group
                  +----------------+
                  |  EC2 Worker    |
                  |    Nodes       |
                  +----------------+
                           |
                           v
                      Kubernetes
                         Pods
```

------------------------------------------------------------------------

# Network Configuration

The project defines:

``` text
VPC CIDR:
10.123.0.0/16
```

Availability Zones:

``` text
us-east-1a
us-east-1b
```

Public subnets:

``` text
10.123.1.0/24
10.123.2.0/24
```

Private subnets:

``` text
10.123.3.0/24
10.123.4.0/24
```

Intra subnets:

``` text
10.123.5.0/24
10.123.6.0/24
```

The VPC configuration enables a NAT Gateway:

``` hcl
enable_nat_gateway = true
```

This allows resources in private subnets to reach external services when
required without directly exposing those resources to the public
internet.

------------------------------------------------------------------------

# EKS Configuration

The configured cluster name is:

``` text
kastro-eks-cluster
```

The cluster is deployed in:

``` text
us-east-1
```

The EKS control-plane endpoint is configured for public access:

``` hcl
cluster_endpoint_public_access = true
```

## EKS Add-ons

The project configures:

-   CoreDNS
-   kube-proxy
-   VPC CNI

These are important components for normal Kubernetes networking and DNS
functionality.

------------------------------------------------------------------------

# Managed Node Group

The repository creates an EKS managed node group:

``` text
cluster-wg
```

Configuration:

``` text
Minimum nodes: 1
Desired nodes: 1
Maximum nodes: 2
Instance type: t3.large
Capacity type: SPOT
```

The configuration uses Spot capacity to reduce compute cost, but Spot
instances can be interrupted by AWS. This is suitable for learning,
development, and selected workloads rather than workloads requiring
uninterrupted capacity.

------------------------------------------------------------------------

# How the Project Works

## Step 1: Engineer pushes Terraform code

Terraform files are stored in GitHub.

``` text
GitHub
   |
   v
provider.tf
vpc.tf
eks.tf
```

## Step 2: Jenkins checks out the repository

The Jenkins pipeline retrieves the source code.

## Step 3: Terraform initialization

Jenkins executes:

``` bash
terraform init
```

This downloads the required Terraform providers and modules.

## Step 4: Terraform validation

Jenkins executes:

``` bash
terraform validate
```

This checks whether the Terraform configuration is syntactically and
structurally valid.

## Step 5: Terraform plan

Jenkins executes:

``` bash
terraform plan
```

The plan shows what AWS resources Terraform intends to create, modify,
or destroy.

## Step 6: Manual approval

The pipeline pauses and asks:

``` text
Approve?
```

The engineer reviews the Terraform plan and approves the deployment.

## Step 7: Apply or destroy

The selected Jenkins parameter controls the operation:

``` text
apply
```

or:

``` text
destroy
```

The pipeline executes:

``` bash
terraform apply --auto-approve
```

or:

``` bash
terraform destroy --auto-approve
```

------------------------------------------------------------------------

# Prerequisites

Before starting, you should have:

-   AWS account
-   IAM permissions required to create the infrastructure
-   Ubuntu EC2 instance for Jenkins
-   GitHub repository
-   Jenkins installed and running
-   Java
-   Terraform
-   AWS CLI
-   kubectl
-   Internet connectivity from the Jenkins host

Recommended Jenkins host for a learning POC:

``` text
Ubuntu 22.04
t2.medium or equivalent
```

Actual Jenkins sizing depends on workload.

------------------------------------------------------------------------

# AWS Requirements

## IAM

The Jenkins execution identity needs permission to provision the
resources required by Terraform.

For a learning project, broad permissions may be used temporarily, but
for a real environment you should use least-privilege IAM policies.

Do not hard-code:

``` text
AWS Access Key
AWS Secret Access Key
```

inside Terraform files, Jenkinsfiles, GitHub repositories, or shell
scripts.

------------------------------------------------------------------------

# Jenkins Setup

Install Jenkins on the Jenkins EC2 instance.

After installation, verify:

``` bash
sudo systemctl status jenkins
```

Jenkins normally listens on:

``` text
8080
```

Open the Jenkins UI from your browser:

``` text
http://<JENKINS-SERVER-IP>:8080
```

Make sure the AWS credentials are stored in Jenkins Credentials rather
than being written directly in the pipeline.

------------------------------------------------------------------------

# Required Jenkins Configuration

Create credentials for AWS.

Example credential IDs:

``` text
AWS_Access_Key
AWS_Secret_Key
```

The pipeline references those credentials.

Install the Jenkins plugins required for pipeline execution and stage
visualization as appropriate for your Jenkins installation.

------------------------------------------------------------------------

# Terraform Configuration

Move into the Terraform directory:

``` bash
cd terraform
```

Initialize Terraform:

``` bash
terraform init
```

Validate:

``` bash
terraform validate
```

Review:

``` bash
terraform plan
```

Apply manually only when testing outside Jenkins:

``` bash
terraform apply
```

Destroy:

``` bash
terraform destroy
```

For this project, Jenkins is intended to perform the apply/destroy
operation.

------------------------------------------------------------------------

# Jenkins Pipeline

A simplified version of the pipeline flow is:

``` groovy
pipeline {
    agent any

    environment {
        AWS_ACCESS_KEY_ID = credentials('AWS_Access_Key')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_Secret_Key')
        AWS_DEFAULT_REGION = 'us-east-1'
    }

    stages {

        stage('Clone the Code') {
            steps {
                checkout scm
            }
        }

        stage('Terraform Initialization') {
            steps {
                dir('terraform') {
                    sh 'terraform init'
                }
            }
        }

        stage('Terraform Validation') {
            steps {
                dir('terraform') {
                    sh 'terraform validate'
                }
            }
        }

        stage('Infrastructure Checks') {
            steps {
                dir('terraform') {
                    sh 'terraform plan'
                }

                input message: 'Approve?', ok: 'Proceed'
            }
        }

        stage('Create/Destroy EKS Cluster') {
            steps {
                dir('terraform') {
                    sh "terraform ${action} --auto-approve"
                }
            }
        }
    }
}
```

> The exact Jenkins syntax should be adapted to the Jenkins credential
> type and agent environment being used.

------------------------------------------------------------------------

# Jenkins Parameter

Configure the Jenkins job as parameterized.

Create a:

``` text
Choice Parameter
```

Name:

``` text
action
```

Choices:

``` text
apply
destroy
```

This provides a simple deployment control.

------------------------------------------------------------------------

# Running the Project

## 1. Open Jenkins

Open the Jenkins dashboard.

## 2. Open the project job

Select the Jenkins pipeline job.

## 3. Select Build with Parameters

Choose:

``` text
action = apply
```

## 4. Start the build

Jenkins will:

``` text
Checkout
   ↓
Terraform Init
   ↓
Terraform Validate
   ↓
Terraform Plan
   ↓
Approval
   ↓
Terraform Apply
```

## 5. Monitor the build

Review the Jenkins console output.

A successful Terraform apply should show resource creation and a
successful completion message.

------------------------------------------------------------------------

# Verify the EKS Cluster

Open the AWS Console.

Navigate to:

``` text
Amazon EKS
```

Find:

``` text
kastro-eks-cluster
```

Check:

-   Cluster status
-   Kubernetes version
-   Networking
-   Add-ons
-   Compute
-   Node group status

Also verify the VPC and related resources.

------------------------------------------------------------------------

# Connect to EKS with kubectl

Install and configure AWS CLI on the client machine.

Verify:

``` bash
aws --version
```

Configure credentials using your preferred secure AWS authentication
method.

Then run:

``` bash
aws eks update-kubeconfig \
  --region us-east-1 \
  --name kastro-eks-cluster
```

Verify the Kubernetes context:

``` bash
kubectl config current-context
```

Check nodes:

``` bash
kubectl get nodes
```

Check all pods:

``` bash
kubectl get pods -A
```

------------------------------------------------------------------------

# Test Kubernetes

Create an nginx pod:

``` bash
kubectl run nginx --image=nginx
```

Check:

``` bash
kubectl get pods
```

Describe it:

``` bash
kubectl describe pod nginx
```

View logs:

``` bash
kubectl logs nginx
```

Delete it:

``` bash
kubectl delete pod nginx
```

------------------------------------------------------------------------

# Destroy the Infrastructure

When the project is no longer required, destroy the resources to avoid
unnecessary AWS charges.

Open Jenkins:

``` text
Build with Parameters
```

Select:

``` text
action = destroy
```

Run the build.

The pipeline executes:

``` bash
terraform destroy --auto-approve
```

Verify that the EKS cluster, node group, VPC, NAT Gateway, and other
Terraform-managed resources have been removed.

If the Jenkins EC2 instance was created only for this project, terminate
it after confirming it is no longer required.

------------------------------------------------------------------------

# Security Recommendations

This repository is a learning/POC project. Before using a similar design
in production:

### 1. Never commit AWS secrets

Do not put credentials in:

``` text
provider.tf
Jenkinsfile
README.md
shell scripts
GitHub Secrets in plain text
```

### 2. Use IAM roles where possible

For EC2-based Jenkins, consider using an IAM instance profile instead of
long-lived access keys.

### 3. Use least privilege

Grant only the permissions required by Terraform.

### 4. Protect the Jenkins server

Restrict port `8080` to trusted networks or use a reverse proxy and
appropriate authentication.

### 5. Protect the EKS API

The current configuration enables public endpoint access. A production
environment should evaluate:

-   private endpoint access
-   restricted public access
-   network controls
-   IAM authentication
-   Kubernetes RBAC

### 6. Protect Terraform state

Terraform state can contain sensitive infrastructure information. Use a
secure remote backend for team/production usage, such as an
appropriately protected S3 backend with state locking support.

------------------------------------------------------------------------

# Cost Considerations

AWS resources created by this project may incur charges.

Potential cost-generating resources include:

-   EKS
-   EC2 worker nodes
-   NAT Gateway
-   EBS volumes
-   Public IPv4 addresses
-   Data transfer
-   Load balancers if later added

For a learning environment, destroy the infrastructure after testing.

Use:

``` text
Jenkins → Build with Parameters → destroy
```

Then verify the AWS Console for remaining resources.

------------------------------------------------------------------------

# Troubleshooting

## Terraform command not found

Check:

``` bash
terraform --version
```

If Terraform is missing, install it and verify the binary is in `PATH`.

------------------------------------------------------------------------

## AWS credentials error

Check:

``` bash
aws sts get-caller-identity
```

If this fails, verify the AWS authentication configuration and IAM
permissions.

------------------------------------------------------------------------

## Terraform AccessDenied

An error such as:

``` text
AccessDenied
```

means the AWS identity used by Terraform does not have the required
permission.

Check:

``` bash
aws sts get-caller-identity
```

Then review the IAM policies attached to the execution identity.

------------------------------------------------------------------------

## kubectl cannot connect to EKS

Run:

``` bash
aws eks update-kubeconfig \
  --region us-east-1 \
  --name kastro-eks-cluster
```

Then:

``` bash
kubectl get nodes
```

Also verify that the cluster exists and that the current AWS identity is
authorized to access it.

------------------------------------------------------------------------

## Jenkins cannot execute Terraform

Verify:

``` bash
terraform --version
java --version
```

Also check the Jenkins service environment and PATH.

------------------------------------------------------------------------

## Jenkins cannot access AWS

Verify the credentials configured in Jenkins.

Also verify:

``` bash
aws sts get-caller-identity
```

from the Jenkins execution environment.

------------------------------------------------------------------------

## EKS nodes are not Ready

Check:

``` bash
kubectl get nodes
kubectl describe nodes
```

Check the EKS node group in the AWS Console.

Also review:

``` bash
kubectl get pods -A
```

for networking or system pod problems.

------------------------------------------------------------------------

# Useful Commands

## AWS

``` bash
aws --version
aws sts get-caller-identity
aws eks list-clusters --region us-east-1
```

## Terraform

``` bash
terraform --version
terraform init
terraform fmt
terraform validate
terraform plan
terraform apply
terraform destroy
terraform show
terraform state list
```

## Kubernetes

``` bash
kubectl version --client
kubectl config get-contexts
kubectl config current-context
kubectl get nodes
kubectl get pods
kubectl get pods -A
kubectl describe pod <pod-name>
kubectl logs <pod-name>
kubectl delete pod <pod-name>
```

------------------------------------------------------------------------

# Terraform Files Explained

## provider.tf

Defines project-level configuration such as:

``` text
AWS region
cluster name
VPC CIDR
availability zones
subnet CIDRs
tags
```

## vpc.tf

Uses the AWS VPC Terraform module to create:

``` text
VPC
Public Subnets
Private Subnets
Intra Subnets
NAT Gateway
Subnet tags
```

## eks.tf

Uses the AWS EKS Terraform module to create:

``` text
EKS Cluster
EKS Add-ons
Managed Node Group
Worker node configuration
Cluster tags
```

------------------------------------------------------------------------

# CI/CD / Infrastructure Flow

This project demonstrates an Infrastructure as Code delivery pipeline:

``` text
                SOURCE
                  |
                  v
              GitHub
                  |
                  v
             Jenkins
                  |
                  v
          Terraform Init
                  |
                  v
        Terraform Validate
                  |
                  v
          Terraform Plan
                  |
                  v
        Manual Approval
                  |
            +-----+-----+
            |           |
          apply       destroy
            |           |
            v           v
        AWS EKS      AWS Cleanup
            |
            v
       Kubernetes
```

This approach makes infrastructure provisioning repeatable and reduces
the need for manually creating AWS resources through the console.

------------------------------------------------------------------------

# Production Improvements

For a production-grade implementation, consider adding:

## Remote Terraform State

Use a secure remote state backend instead of local state.

Example architecture:

``` text
Jenkins
   |
   v
Terraform
   |
   +--> S3 Terraform State
   |
   +--> AWS Infrastructure
```

## Terraform Variables

Move hard-coded values into:

``` text
variables.tf
terraform.tfvars
```

Examples:

``` text
region
cluster_name
vpc_cidr
instance_type
desired_capacity
```

## Separate Environments

Create:

``` text
dev
qa
staging
prod
```

using Terraform workspaces or, preferably for larger environments,
separate configurations/state.

## Validation and Security Scanning

Add tools such as:

``` text
tflint
tfsec / Trivy
Checkov
```

to the Jenkins pipeline.

## Approval Controls

Use stronger approval controls for production.

## Notifications

Integrate Jenkins with:

``` text
Email
Slack
Microsoft Teams
```

to notify teams about deployment results.

## Monitoring

Add:

``` text
CloudWatch
Prometheus
Grafana
```

for infrastructure and Kubernetes monitoring.

------------------------------------------------------------------------

# Suggested Future Repository Structure

A more scalable version of this project could use:

``` text
Jenkins-Terraform-EKS/
│
├── README.md
├── INDEX.md
├── Jenkinsfile
│
├── terraform/
│   ├── versions.tf
│   ├── provider.tf
│   ├── variables.tf
│   ├── outputs.tf
│   ├── vpc.tf
│   ├── eks.tf
│   └── terraform.tfvars.example
│
├── docs/
│   ├── architecture.md
│   ├── deployment.md
│   ├── troubleshooting.md
│   └── security.md
│
└── scripts/
    ├── install-tools.sh
    └── cleanup.sh
```

This structure separates infrastructure, pipeline configuration,
documentation, and operational scripts.

------------------------------------------------------------------------

# Learning Outcomes

After completing this project, you should understand:

### AWS

-   VPC
-   Subnets
-   NAT Gateway
-   EKS
-   EC2
-   IAM
-   AWS CLI

### Terraform

-   Providers
-   Modules
-   Variables
-   Resources
-   State
-   Init
-   Validate
-   Plan
-   Apply
-   Destroy

### Jenkins

-   Pipeline jobs
-   Credentials
-   Environment variables
-   Build parameters
-   Pipeline stages
-   Manual approval
-   Console logs

### Kubernetes

-   EKS
-   Nodes
-   Pods
-   kubectl
-   Kubernetes contexts
-   Basic workload deployment

### DevOps

-   Git-based infrastructure management
-   Infrastructure as Code
-   Automated provisioning
-   Approval-based deployment
-   Infrastructure cleanup

------------------------------------------------------------------------

# Project Benefits

This project helps demonstrate how an organization can move from:

``` text
Manual AWS Console
       ↓
Manual infrastructure creation
       ↓
Configuration inconsistency
```

toward:

``` text
GitHub
   ↓
Jenkins
   ↓
Terraform
   ↓
AWS
   ↓
EKS
```

The infrastructure becomes version-controlled, repeatable, reviewable,
and automatable.

------------------------------------------------------------------------

# Important Notes

1.  The repository currently contains the Terraform configuration and
    setup documentation, but the Jenkins pipeline code is documented in
    `Creation of EKS Cluster.txt` rather than stored as a `Jenkinsfile`.
2.  The AWS region configured in the Terraform files is `us-east-1`.
    Keep the Jenkins `AWS_DEFAULT_REGION` and EKS commands consistent
    with that region.
3.  The node group uses Spot capacity, so node interruption is possible.
4.  The EKS endpoint is configured for public access.
5.  AWS resources can generate charges. Destroy unused infrastructure
    promptly.
6.  Never commit AWS credentials or other secrets to GitHub.

------------------------------------------------------------------------

# Conclusion

This project provides a practical example of automating Amazon EKS
infrastructure with Jenkins and Terraform.

The core workflow is:

``` text
GitHub
  ↓
Jenkins
  ↓
Terraform Init
  ↓
Terraform Validate
  ↓
Terraform Plan
  ↓
Manual Approval
  ↓
Terraform Apply
  ↓
AWS VPC + EKS
  ↓
Kubernetes
```

For cleanup:

``` text
Jenkins
  ↓
Build with Parameters
  ↓
action = destroy
  ↓
Terraform Destroy
  ↓
AWS Resources Removed
```

It is a useful foundation for learning **AWS + Terraform + Jenkins +
Kubernetes** and can be extended into a more complete DevOps platform
with remote state, security scanning, monitoring, notifications,
environment management, and production-grade access controls.

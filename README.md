#  Infrastructure as Code – Terraform EKS Deployment

This repository contains the Terraform code used to provision a complete AWS infrastructure for deploying a Kubernetes-based application using **Amazon EKS**.

---
## Architectural diagram
![](/img/architectural%20setup.png)
*NB: Please click this [link](https://github.com/Olasunkanmi-O/vprofile-action/raw/main/img/app-deploy.drawio.png) to see the architectural diagram of the detailed action activity of the workflow*

##  What It Provisions

This Terraform setup provisions the following AWS resources:

- **VPC**
- **Public and Private Subnets**
- **Route Tables**
- **Internet Gateway**
- **NAT Gateway**
- **DHCP Options**
- **Tags**
- **EKS Cluster**
- **EKS Managed Node Groups**
- **Security Groups**  
  - Cluster Security Group  
  - Node Security Group

---

##  Requirements

Ensure the following are installed and configured:

- [Terraform CLI](https://developer.hashicorp.com/terraform/downloads) v1.6.3
- [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)
- A backend (e.g., S3) for storing the remote Terraform state
- A GitHub account for storing your Terraform code and workflows

---

##  Terraform Commands

```bash
terraform init
terraform fmt -check
terraform validate
terraform plan -out=planfile
terraform apply -auto-approve -input=false -parallelism=1 planfile
```

## Repository Setup 
1.  Create a New Repository
 - On GitHub, create a new repository for your Terraform code.
2. Generate an SSH key  
```bash 
ssh-keygen -t rsa -b 4096 -f ~/.ssh/<keyname>
```
- Add the public key (~/.ssh/<keyname>.pub) to your GitHub SSH keys.
- Export the private key for Git usage:
```bash 
export GIT_SSH_COMMAND="ssh -i ~/.ssh/<keyname>"
```
3. clone and configure Git
```bash
git clone git@github.com:<your-username>/<your-repo>.git
cd <your-repo>
git config core.sshCommand "ssh -i ~/.ssh/<keyname> -F /dev/null"
``` 
## AWS IAM Setup
1. create an IAM user with below accesses, and create access key
 - `AmazonEC2FullAccess`
 - `AmazonRoute53FullAccess`
 - `AmazonS3FullAccess`
 - `IAMFullAccess`
 - `AmazonVPCFullAccess`
 - `AmazonSQSFullAccess`
 - `AmazonEventBridgeFullAccess`

2. store the access key and secret access key as Github Secrets:

| Secret Name | Description |
| --- | --- |
| `AWS_ACCESS_KEY_ID` | IAM user access key ID |
| `AWS_SECRET_ACCESS_KEY` | IAM user secret access key |
| `BUCKET_TF_STATE` | Name of your S3 bucket for state |

---

## Remote State Backend

```hcl
terraform {
  backend "s3" {
    bucket = "your-bucket-name"
    key    = "terraform.tfstate"
    region = "us-east-1"
  }
}

```


Key terraform files 
 - [`main.tf`](/terraform/main.tf)
 - [`vpc.tf`](/terraform/vpc.tf)
 - [`eks-cluster.tf`](/terraform/eks-cluster.tf)  
 - [`variables.tf`](/terraform/variables.tf)
 - [`output.tf`](/terraform/outputs.tf)

## Github Action Workflow
1. Directory Structure
- Create a `.github/workflows/` root  directory in the root of your repository.

![](/img/folder-structure-iac.png)

```yaml
name: "Terraform Workflow"

on:
  push:
    branches:
      - main
      - stage
    paths:
      - terraform/**
  pull_request:
    branches:
      - main
    paths:
      - terraform/**

```
Push Trigger: Runs when code is pushed to `main` or `stage`, and if files under `terraform/` change.

Pull Request Trigger: Runs on PRs targeting `main` that affect `terraform/`.

## Safe Deployment Practice
To prevent accidental infrastructure changes, the `terraform apply` step in your workflow is gated:
- Only runs after merging changes from `stage` → `main`.
- Ensures changes are peer-reviewed before affecting real infrastructure.

![](/img/apply.png)

## confirm infrastructure build
![](/img/eks-cluster.png)
![](/img/node-grps.png)
![](/img/nodes.png)



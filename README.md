#  VProfile Infrastructure as Code (IaC)

[](https://www.google.com/search?q=https://github.com/YOUR_USERNAME/YOUR_REPO/actions/workflows/vprofile-iac.yml)

This repository contains the complete Terraform configuration for deploying and managing the **VProfile** application infrastructure on Amazon Web Services (AWS). All infrastructure changes are managed, reviewed, and deployed via a robust **GitHub Actions CI/CD pipeline** (defined in `.github/workflows/vprofile-iac.yml`).

## Architectural diagram
![](/img/architectural%20setup.png)

##  Project Overview

This IaC setup provisions a fully functioning, secure, and scalable **Kubernetes environment (EKS)**, including the network and compute layers necessary to run the VProfile microservices application.

### Major Components Deployed:

  * **Networking (VPC):** A dedicated VPC (`vprofile-eks`) spanning three Availability Zones with both public and private subnets.
  * **Compute:** A high-availability **AWS EKS Cluster** (`vprofile-eks`) running Kubernetes version `1.27`.
  * **Worker Nodes:** Two separate, EKS-managed node groups configured for redundancy and scaling.
  * **State Backend:** Secure S3 backend for remote state management and state locking.

-----

##  Infrastructure Configuration Details

### 1\. Networking (`module.vpc`)

The networking is provisioned using the official `terraform-aws-modules/vpc/aws` module, establishing a secure network boundary.

| Configuration Detail | Value/Description |
| :--- | :--- |
| **VPC CIDR** | `172.20.0.0/16` |
| **Availability** | Utilizes the first **3 Availability Zones** in the region. |
| **Subnets** | 3 Private Subnets (`172.20.1.0/24`, etc.) and 3 Public Subnets. |
| **NAT Gateway** | Enabled (`single_nat_gateway = true`), providing outbound-only internet access for all resources in private subnets. |
| **EKS Tagging** | Tags subnets for automatic discovery by EKS components (Load Balancers, etc.). |

### 2\. EKS Cluster (`module.eks`)

The Kubernetes control plane and worker nodes are managed using the official `terraform-aws-modules/eks/aws` module.

| Configuration Detail | Value/Description |
| :--- | :--- |
| **Cluster Name** | `vprofile-eks` (set via `local.cluster_name`) |
| **Kubernetes Version** | `1.27` |
| **Access** | `cluster_endpoint_public_access = true` (Public API endpoint enabled). |
| **VPC Integration** | Cluster control plane is associated with the provisioned VPC and private subnets. |

#### **EKS Managed Node Groups**

The infrastructure includes two separate managed node groups for application redundancy and potential workload separation:

| Node Group | Instance Type | Min Size | Desired Size | Max Size |
| :--- | :--- | :--- | :--- | :--- |
| **`node-group-1` (one)** | `t3.small` | 1 | 2 | 3 |
| **`node-group-2` (two)** | `t3.small` | 1 | 1 | 2 |

### 3\. Terraform Backend and Providers

The project enforces version consistency and secure state management:

| Component | Detail |
| :--- | :--- |
| **Required Version** | `1.12.1` |
| **Remote Backend** | **S3** bucket named `probucket-file` in `us-east-1` with state file `terraform.tfstate`. |
| **Required Providers** | `aws (~> 5.25.0)`, `kubernetes (~> 2.23.0)`, `random`, `tls`, and `cloudinit`. |

-----

##  CI/CD Pipeline (GitHub Actions)

The core workflow (defined in `.github/workflows/vprofile-iac.yml`) ensures that infrastructure is consistent, auditable, and fully managed through code.

### Workflow Triggers

| Event | Branch | Path | Action |
| :--- | :--- | :--- | :--- |
| **`push`** | `main`, `stage` | `terraform/**` | Full CI/CD (Plan, Apply for `main`) |
| **`pull_request`** | `main` | `terraform/**` | **Validation & Plan only** (Safety Review) |

### Execution Steps

1.  **Code Validation:** Runs `fmt -check` and `validate`.
2.  **Plan:** Executes `terraform plan` on every push/PR to show exact infrastructure changes.
3.  **Apply (Gated):** Executes `terraform apply -auto-approve` **only on a successful push to the `main` branch.**
4.  **Post-Deployment Configuration:**
      * Configures AWS credentials.
      * Retrieves EKS `kubeconfig`.
      * Deploys the NGINX Ingress Controller using `kubectl apply`.

###  Security & Secrets

All AWS credentials (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`) and the `BUCKET_TF_STATE` are securely injected into the workflow using **GitHub Repository Secrets**.


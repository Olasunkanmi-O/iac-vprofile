# Terraform code 

##  What It Provisions
- VPC
- Subnets (public/private)
- NAT Gateway(s)
- Route tables
- Internet Gateway
- DHCP options
- Tags
- EKS Cluster
- EKS Managed Node Groups
- Security Groups - Cluster Security Group and Node security Group

##  Requirements
- Terraform CLI v1.x
- AWS CLI configured with appropriate credentials
- Backend for remote state (optional)
- Terraform version 1.6.3


### Steps
* terraform init
* terraform fmt -check
* terraform validate
* terraform plan -out planfile
* terraform apply -auto-approve -input=false -parallelism=1 planfile


Prepare the repository
1. craete a new repository in your github account
2. create an ssh key in this path on your local machine ```~/.ssh/<keyname>```
3. copy the public key to the github account and add to ssh key in the github account
4. export the private key in your local machine 

    ```
    export GIT_SSH_COMMAND="ssh -i ~/.ssh/<keyname>"
    ```
5. make a directory in your local machine where to clone the newly created repo
6. clone the repo and cd into it.
7. to ensure the repository always use the SSH key use below command
    ```bash
    git config core. sshCommand "ssh -i ~/.ssh/<keyname> -F /dev/null"
    ```
8. create an IAM user with below accesses, and create access key
 - AmazonEC2FullAccess
 - AmazonRoute53FullAccess
 - AmazonS3FullAccess
 - IAMFullAccess
 - AmazonVPCFullAccess
 - AmazonSQSFullAccess
 - AmazonEventBridgeFullAccess
9. copy the access key and secret access key to secrets in github with below variables
 - AWS_ACCESS_KEY_ID
 - AWS_SECRET_ACCESS_KEY
10. create S3 bucket to be used for terraform state, store your bucket name in github secret as well, variable BUCKET_TF_STATE
11. write your terraform code, major files are below
 - [main.tf](/terraform/main.tf)
 - [vpc.tf](/terraform/vpc.tf)
 - [eks-cluster.tf](/terraform/eks-cluster.tf)  
12. create a workflows directory structure at the parent level with the exact name
![](/img/folder-structure-iac.png)
13. create a file (yaml), this is where we will write the workflows. check this [file](.github\workflows\terraform.yml) 
14. My workflow trigger as shown is defined below
![](/img/workflow.png)

a. push:
- Triggered when there is a push (commit or merge) to either: main or stage
- Only if the changes affect files inside the terraform/ directory (or its sub-folders)

b. pull_request:
- Triggered when a pull request targets the main branch
- Only if the pull request includes changes in the terraform/ directory (or its sub-folders)
15. To trigger the terraform apply which will build the infrastructure, below condition was attached to the step to avoid accidental build, since all changes will be done on the stage branch, until there is a merge to main there wouldn't be any infrastructure build
![](/img/apply.png)
16. 
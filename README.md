# Terraform Remote State with S3

Configuring Terraform remote state storage on AWS S3 to enable team collaboration and safe state management for infrastructure-as-code workflows.

---

## Project at a Glance

| | |
|---|---|
| **Tools Used** | Terraform · AWS S3 · AWS EC2 |
| **Platform** | AWS (us-east-1) |
| **Languages** | HCL (Terraform) |
| **What It Does** | Stores Terraform state remotely in S3 so multiple engineers can share and lock infrastructure state safely |

## The Problem This Project Solves

When Terraform runs, it records the current state of your infrastructure in a file called `terraform.tfstate`. By default this file sits on your local machine. That works fine when you are the only person touching the infrastructure — but it breaks immediately the moment a second engineer joins the team. Two people running `terraform apply` against different local state files will cause conflicting changes, duplicate resources, and infrastructure drift.

Remote state solves this by storing the state file in a shared, durable location — in this case an S3 bucket. Every engineer's Terraform points to the same file. S3 provides versioning so you can roll back if a bad apply corrupts the state. In production pipelines, a DynamoDB table is added alongside S3 to provide state locking, preventing two engineers from applying simultaneously.

This project demonstrates the backend configuration pattern that every production Terraform deployment uses. It is one of the first things a DevOps or cloud engineer sets up before building any serious infrastructure.

## Architecture

```
LOCAL MACHINE (Engineer A or B)
       |
       | terraform init / plan / apply
       v
TERRAFORM BACKEND (backend.tf)
       |
       | reads and writes state
       v
AWS S3 BUCKET: shaymill-terrafrom-bucket-2025
  └── myStateFile.tfstate   (shared state file)
       |
       | state describes
       v
AWS INFRASTRUCTURE (main.tf)
  └── EC2 instance: Web Server (t3.micro, us-east-1)
        ├── Security Group: allows HTTP :80 and SSH :22
        └── User data: installs and starts Apache httpd
```

## Repository Structure

```
terraform-remote-state-s3/
├── backend.tf      # S3 remote state backend configuration
├── main.tf         # EC2 instance and security group definitions
└── .gitignore      # excludes .terraform/ and *.tfstate from version control
```

## Key Files Explained

### `backend.tf` — Remote State Configuration

```hcl
terraform {
  backend "s3" {
    key    = "myStateFile.tfstate"
    bucket = "shaymill-terrafrom-bucket-2025"
    region = "us-east-1"
  }
}
```

This tells Terraform: instead of writing state locally, write it to the named S3 bucket. The `key` is the path within the bucket where the file will live. The bucket must exist before running `terraform init`.

### `main.tf` — Infrastructure Definition

Provisions a single EC2 instance running Apache httpd. The instance is bootstrapped via user data — no manual SSH required. A security group allows inbound HTTP (port 80) and SSH (port 22) traffic.

## How to Reproduce

**Prerequisites:** AWS CLI configured, Terraform installed, S3 bucket created

```bash
# 1. Clone the repository
git clone https://github.com/seyifalode-cmd/terraform-remote-state-s3.git
cd terraform-remote-state-s3

# 2. Update backend.tf with your own S3 bucket name
#    Edit: bucket = "your-bucket-name"

# 3. Initialize — this connects Terraform to S3
terraform init

# 4. Preview changes
terraform plan

# 5. Apply infrastructure
terraform apply

# 6. Verify state is stored in S3
aws s3 ls s3://your-bucket-name/

# 7. Destroy when done
terraform destroy
```

## Why This Matters

Remote state is not optional in team environments. Without it, infrastructure management becomes chaotic — teams resort to one-person-at-a-time conventions, state files sent via email, or worse, rebuilding from scratch after state is lost. S3 backend is the industry standard starting point before adding DynamoDB locking for full concurrency safety.

---

*Oluwaseyi Michael Falode · Cybersecurity & Cloud Security Engineer · Toronto, ON*

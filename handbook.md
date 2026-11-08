# AWS SRE & Platform Engineering Capstone: End-to-End Terraform & Observability Handbook

## 📋 Table of Contents

1. [Project Overview & Engineering Philosophy](https://www.google.com/search?q=%231-project-overview--engineering-philosophy)
2. [Prerequisites & Tooling Setup](https://www.google.com/search?q=%232-prerequisites--tooling-setup)
3. [Step 1: Provider & Backend Configuration (`providers.tf`)](https://www.google.com/search?q=%233-step-1-provider--backend-configuration-providerstf)
4. [Step 2: Network Infrastructure Provisioning (`vpc.tf`)](https://www.google.com/search?q=%234-step-2-network-infrastructure-provisioning-vpctf)
5. [Step 3: EKS Control Plane & Worker Nodes (`eks.tf`)](https://www.google.com/search?q=%235-step-3-eks-control-plane--worker-nodes-ekstf)
6. [Step 4: Terraform Lifecycle Operations (`init`, `plan`, `apply`)](https://www.google.com/search?q=%236-step-4-terraform-lifecycle-operations-init-plan-apply)
7. [Step 5: Observability & APM Integration (Dynatrace & Dynakube)](https://www.google.com/search?q=%237-step-5-observability--apm-integration-dynatrace--dynakube)
8. [Step 6: FinOps & Safe Teardown Workflow](https://www.google.com/search?q=%238-step-6-finops--safe-teardown-workflow)

---

## 1. Project Overview & Engineering Philosophy

* **What we did:** Built and tore down a production-grade Amazon EKS (v1.31) cluster in `ap-south-1` (Mumbai) entirely through **Terraform Infrastructure as Code (IaC)**, integrating Dynatrace APM observability.
* **Why we did it:** Manual provisioning in the AWS Console is prone to human error, lacks version control, and leaves "ghost resources." Using Terraform ensures reproducibility, auditing, and clean state management.

---

## 2. Prerequisites & Tooling Setup

Before writing code, ensure your local environment has the required binaries installed and configured:

* **AWS CLI:** Configured with administrative or appropriate least-privilege IAM credentials (`aws configure`).
* **Terraform:** Installed (v1.x) for HCL-based resource provisioning.
* **Kubernetes CLI (`kubectl`):** For interacting with the cluster after provisioning.
* **Git:** For version control.

---

## 3. Step 1: Provider & Backend Configuration (`providers.tf`)

* **What we do:** Tell Terraform which cloud provider to talk to, which region to deploy into, and lock down required provider versions.
* **Why we do it:** This acts as the entry point for Terraform to download the necessary AWS and Kubernetes plugins.
* **Code Implementation (`providers.tf`):**
```hcl
terraform {
  required_version = ">= 1.0.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    kubernetes = {
      source  = "hashicorp/kubernetes"
      version = "~> 2.20"
    }
  }
}

provider "aws" {
  region = "ap-south-1"
}

```



---

## 4. Step 2: Network Infrastructure Provisioning (`vpc.tf`)

* **What we do:** Declare a custom Virtual Private Cloud (VPC), public subnets, private subnets, Internet Gateways, and route tables.
* **Why we do it:** EKS worker nodes and control planes require a secure, isolated network topology with proper routing for external access and internal microservices communication.
* **Code Implementation (`vpc.tf` summary):**
```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.0.0"

  name = "sre-observability-vpc"
  cidr = "10.0.0.0/16"

  azs             = ["ap-south-1a", "ap-south-1b"]
  public_subnets  = ["10.0.1.0/24", "10.0.2.0/24"]
  private_subnets = ["10.0.101.0/24", "10.0.102.0/24"]

  enable_nat_gateway   = true
  single_nat_gateway   = true
  enable_dns_hostnames = true
}

```



---

## 5. Step 3: EKS Control Plane & Worker Nodes (`eks.tf`)

* **What we do:** Define the Amazon EKS cluster resource (`sre-observability-cluster`) running Kubernetes v1.31, along with managed node groups (`sre_nodes`) backed by `t3.micro` instances.
* **Why we do it:** Fully automates control-plane deployment and worker node registration without touching the AWS Web Console.
* **Code Implementation (`eks.tf` summary):**
```hcl
module "eks" {
  source          = "terraform-aws-modules/eks/aws"
  cluster_name    = "sre-observability-cluster"
  cluster_version = "1.31"
  subnet_ids      = module.vpc.private_subnets
  vpc_id          = module.vpc.vpc_id

  eks_managed_node_groups = {
    sre_nodes = {
      min_size     = 1
      max_size     = 3
      desired_size = 2
      instance_types = ["t3.micro"]
    }
  }
}

```



---

## 6. Step 4: Terraform Lifecycle Operations (`init`, `plan`, `apply`)

* **What we did:** Executed local terminal commands to initialize, preview, and provision the infrastructure.
* **Why we did it:** Validates syntax, downloads provider plugins, and generates the `terraform.tfstate` tracking file.
* **Baby-Step Terminal Workflow:**
1. Initialize working directory:
```cmd
terraform init

```


2. Preview exact changes before applying (Plan phase):
```cmd
terraform plan -out=tfplan

```


3. Execute resource creation:
```cmd
terraform apply tfplan

```





---

## 7. Step 5: Observability & APM Integration (Dynatrace & Dynakube)

* **What we did:** Configured the `dynakube` Custom Resource Definition (CRD) manifest inside the cluster to deploy the Dynatrace OneAgent across all nodes.
* **Why we do it:** Captures deep application telemetry, logs, and infrastructure metrics out-of-the-box for distributed tracing and SRE monitoring.
* **How we did it:** Applied the declarative `dynakube` YAML configuration against the active EKS cluster context via `kubectl`.

---

## 8. Step 6: FinOps & Safe Teardown Workflow

* **What we did:** Safely dismantled the entire infrastructure to prevent idle cloud billing.
* **Issues Faced & Solutions:**
* Direct cluster deletion attempts can trigger `ResourceInUseException` if node groups or active EC2 instances are still terminating.
* **The Professional Resolution:** Managed node groups were drained/terminated first via AWS CLI/Terraform, and final status checks (`ResourceNotFoundException`) confirmed zero orphaned assets.

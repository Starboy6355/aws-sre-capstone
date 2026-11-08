# AWS SRE & Observability Capstone Project

A production-grade Kubernetes observability and infrastructure lifecycle management project deployed on Amazon EKS. Designed to demonstrate end-to-end cloud platform engineering, automated telemetry, and rigorous cost governance.

---

## 🏗️ Architecture & Tech Stack
* **Cloud Provider:** AWS (`ap-south-1` / Mumbai)
* **Orchestration:** Amazon EKS (Kubernetes v1.31) with managed node groups (`t3.micro`)
* **Networking:** Custom VPC, public/private subnets, security groups, and Internet Gateways
* **Observability & APM:** Dynatrace, OpenTelemetry (YAML configurations), and ELK stack concepts
* **Tooling:** AWS CLI, Kubernetes CLI (`kubectl`), Git / GitHub

---

## 🔄 Project Lifecycle & SRE Operations

### 1. Provisioning & Deployment
* Deployed a managed control plane via Amazon EKS (`sre-observability-cluster`).
* Configured node groups utilizing Amazon Linux 2023 with Auto Scaling group parameters.
* Established secure cluster access and node-to-control-plane communication.

### 2. Observability Framework Integration
* Deployed monitoring components and telemetry collectors into isolated Kubernetes namespaces (`dynatrace`).
* Configured OpenTelemetry pipelines to capture infrastructure and application-level metrics, simulating real-world production monitoring setup.

### 3. Automated Teardown & Cost Governance (FinOps)
To eliminate idle cloud waste and prevent unexpected billing, a structured CLI-based decommissioning workflow was executed:
* **Workload Isolation:** Removed monitoring agents and application namespaces.
* **Node Group Decommissioning:** Executed `aws eks delete-nodegroup` to drain and trigger Auto Scaling Group termination.
* **Compute Termination:** Verified EC2 worker node states (`shutting-down` $\rightarrow$ terminated).
* **Control Plane Deletion:** Executed `aws eks delete-cluster` and confirmed complete resource removal via `ResourceNotFoundException` validations.

---

## 💡 Interview Talking Points
* **Why EKS & Kubernetes v1.31?** Demonstrates familiarity with modern managed Kubernetes lifecycles, minor version upgrades, and control-plane decoupling.
* **The Importance of FinOps in SRE:** Highlight how automated CLI teardowns prevent "ghost resource" charges (such as lingering NAT Gateways or unattached EKS nodes) during non-production cycles.
* **Observability First:** Emphasize how integrating Dynatrace and OpenTelemetry early ensures platform reliability and rapid MTTR (Mean Time to Resolution) in distributed microservices environments.

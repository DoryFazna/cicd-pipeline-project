# The Ultimate CICD Corporate DevOps Pipeline Project | Real-Time DevOps Project

This project demonstrates how to build a real-world **Corporate DevOps pipeline** using **Jenkins**, **Kubernetes**, **SonarQube**, and **Nexus** as part of a comprehensive **CI/CD** workflow. The goal is to create a fully automated and secure **CI/CD pipeline** for corporate environments, integrating code repositories, build automation, continuous testing, and deployment.

## Project Overview:
This project is based on a detailed course, and it's divided into **several key phases** that cover everything from **infrastructure setup** to **monitoring**:

### Phase 1: Setup Infrastructure (Kubernetes Cluster)
- **VPC**: Created a **VPC** for hosting the infrastructure.
- **Security Groups**: Set up necessary **AWS security groups** to allow communication between instances.
- **EC2 Instances**: Launched 3 **EC2 instances** — **Master**, **Slave-1**, and **Slave-2**.
  - **Master Instance**: For the Kubernetes master node.
  - **Slave-1 & Slave-2 Instances**: For the worker nodes.
- **Kubernetes Dependencies**: Installed all necessary **Kubernetes dependencies** (e.g., Docker, kubeadm, kubelet, kubectl) on all instances.

### Security Scan of Kubernetes Cluster
- Plan to scan the Kubernetes cluster for potential vulnerabilities and ensure security best practices.

### Create VMs for Jenkins, SonarQube, & Nexus
- Next steps will include provisioning **virtual machines** for **Jenkins**, **SonarQube**, and **Nexus** to enable build automation, code quality analysis, and artifact storage.

### Phase 2: Git Repo Setup
- Setting up the **Git repository** to store code and manage versioning.

### Configure Jenkins for CI/CD Automation
- Configuring **Jenkins** for the automation of the build, test, and deployment pipeline.

### CICD Full Stack Pipeline
- Creating a fully functional **CI/CD pipeline** involving code checkout, build, unit testing, deployment, and integration with **SonarQube** for static code analysis.

### Monitoring
- Setting up **monitoring** to track the health of the pipeline and infrastructure.

## Current Status:
- **Phase 1**: VPC, EC2 instances, and Kubernetes dependencies setup are complete.
- **Next Steps**: 
  - Set up Kubernetes cluster (init/master node configuration).
  - Continue with Jenkins and SonarQube integration for CI/CD.

---

## Technologies Used:
- **AWS EC2** for infrastructure provisioning
- **Jenkins** for CI/CD automation
- **SonarQube** for static code analysis
- **Nexus** for artifact storage
- **Kubernetes** for container orchestration
- **Docker** for containerization
- **Terraform/Ansible** for infrastructure as code (optional)

---

### 🚀 Stay tuned for updates as I progress through each phase and complete the pipeline!

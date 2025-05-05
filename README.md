Checkout my blogpost of complete step-by-step explanation : https://connect-ubuntu-as-slave-jenkins.blogspot.com/2025/05/building-production-grade-cicd-pipeline.html

# The Ultimate CICD Corporate DevOps Pipeline Project | Real-Time DevOps Project
![CI/CD Pipeline](images/self-created-pipeline-diagram.jpeg)

This project demonstrates how to build a real-world **Corporate DevOps pipeline** using **Jenkins**, **Kubernetes**, **SonarQube**, and **Nexus** as part of a comprehensive **CI/CD** workflow. The goal is to create a fully automated and secure **CI/CD pipeline** for corporate environments, integrating code repositories, build automation, continuous testing, and deployment.

## Project Overview:
This project is based on a detailed course, and it's divided into **several key phases** that cover everything from **infrastructure setup** to **monitoring**:

### Phase 1: Setup Infrastructure (Kubernetes Cluster)
#### ✅  Configure AWS and create 3 instances
- **VPC**: Created a **VPC** for hosting the infrastructure.
- **Security Groups**: Set up necessary **AWS security groups** to allow communication between instances.
- **EC2 Instances**: Launched 3 **EC2 instances** — **Master**, **Slave-1**, and **Slave-2**.
  - **Master Instance**: For the Kubernetes master node.
  - **Slave-1 & Slave-2 Instances**: For the worker nodes.
- **Kubernetes Dependencies**: Installed all necessary **Kubernetes dependencies** (e.g., Docker, kubeadm, kubelet, kubectl) on all instances.
 
#### ✅  Security Scan of Kubernetes Cluster
- Initialized the Kubernetes cluster using kubeadm.
- Joined worker nodes to the cluster.
- Verified nodes using kubectl get nodes.
- Checked basic security configurations and node roles.
- Ensured the cluster is operational and secured at a basic level.

#### ✅ Create VMs for Jenkins, SonarQube, & Nexus
- Provisioned 3 separate EC2 instances for:
  - Jenkins (CI/CD server)
  - SonarQube (Code quality analysis)
  - Nexus (Artifact repository)
- Installed and setup the tools in respective VMs.

## Phase 2: Git Repo Setup
#### ✅ Initialized and pushed project code to a Git repository. (in this tutorial - boardGame project)

## Current Status:
### 🔄 Phase 3: Configure Jenkins for CI/CD Automation (In Progress)
Currently working on:
- Configuring Jenkins.
- Setting up required plugins and integration with Git repository.
- Defining initial Jenkins jobs for build and test stages.
- Preparing to integrate with SonarQube and Nexus in the pipeline.

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

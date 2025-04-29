# Project Notes: CI/CD Pipeline

## AWS Setup
1. Create a VPC (used the default one)
2. EC2 > Create a Security group with the following inbound rules

| Protocol | Port(s)        | Purpose                                                                 | Notes                                             |
|----------|----------------|-------------------------------------------------------------------------|---------------------------------------------------|
| TCP      | 22             | SSH access to EC2 instances                                             | Required for manual access and debugging          |
| TCP      | 80             | HTTP                                                                    | For serving web applications                      |
| TCP      | 443            | HTTPS                                                                   | Secure traffic for apps/services                  |
| TCP      | 6443           | Kubernetes API Server                                                   | Required to set up the Kubernetes master node     |
| TCP      | 30000-32767    | Kubernetes NodePort range                                               | Used for exposing services via NodePort           |
| TCP      | 3000-10000     | Custom application deployment range                                     | For deploying internal apps or services           |
| TCP      | 465            | SMTPS                                                                   | For Jenkins email notifications via Gmail         |
| TCP      | 25             | SMTP                                                                    | Standard mail (not used in this project)          |


3. Launch EC2 instances
  ### EC2 Instance Setup
  - **Instances:** 3 total  
    - 1 master, 2 workers for Kubernetes cluster
  - **AMI:** Ubuntu (latest LTS)
  - **Instance Type:** `t2.medium` (2 vCPU, 4 GiB RAM)
  - > ⚠️ Note: `t2.micro` is not compatible with Kubernetes. It only provides 1 vCPU, but Kubernetes requires at least 2 vCPUs to initialize the cluster (especially the master node)
  - **Key Pair:**  
    - Create one if not available  
    - Download and store `.pem` file safely  
  - **Security Group:**  
    - Use the one with predefined inbound rules (see Security section)
  - **Storage:** 25 GB (gp2 or gp3)

4. Rename the created instances.
    - Master, Slave-1, Slave-2
  
5. Login to the instances via terminal
  > **Note:**  
  > The tutorial initially recommended installing **MobaXterm** for terminal access, but it only works on Windows.  
  > Since I'm using macOS, I'm using the built-in **Terminal** instead.  
  > To differentiate between the master and worker nodes, I've renamed the terminal sessions and customized the colors for easy identification.

```bash
ssh -i "your-key.pem" ubuntu@<public-ip>
```
  - copy the <public-ip> from the aws instance properties



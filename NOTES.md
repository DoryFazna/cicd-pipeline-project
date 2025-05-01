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

After logging into the instances, follow this script to download necessary dependencies


Then run the command to start the master node
and run the worker node code which joins into the kubernetes cluster

then u can close the slave nodes for now since we wont be needing to run any commands..

kubectl get nodes - to check if nodes are running fine

now youre gonna initiate 3 more instances. 
1- sonarQube
2- Nexus
3- Jenkins

install docker in both nexus and sonarqube instances.

by default only root use can execute docker commands, to make other users enable use docker command
run this command
sudo chmod 666 /var/run/docker.sock


Create a docker container to install sonarqube and nexus

docker run -d --name sonar -p 9000:9000 sonarqube:lts-community

Note: -d means run on detached mode, means run in background
host port 9000 - which means the port thats gonna be open in this virtual machine
container port 9000 - the  port that gets opened in container

docker run -d --name Nexus -p 8081:8081 sonatype/nexus3


check if u can see both apps running by typing the ip address and the port on search

login with admin user admin password and change passwords for sonarqube
login with admin user and get the password from the correct location by accessing the docker container with the following code
- docker exec -it <ContainerID> /bin/bash
-it means access Interactive Terminal (it)
/bin/bash mentions which type of terminal you want to use to interact with


Lets set up Jenkins node
- install java (make sure its jdk17 or above other wise it wil lgive issues later)
- install jenkins
- install docker
- install trivy (whic is a code checker- will be used in pipeline)

go and access jenkins thru the ipaddress and port, and unlock and login to it.

LETS INSTALL PLUGINS

go to manage jenkins (we r gonna install certain pluins we need)
click plugins 
download the following plugins :
- Eclipse tumerin installer - diff jenkins jobs might need diff jdks, when needing multiple versions of jdk, can use this one to setup them for usage
- Maven integration
- config file provider - for maven (which helps to set up config files like settings.xml and more files which are needed)
- Pipeline maven integration
- SonarQube scanner - allows easy integration of sonarqube, an opensource platform for continuous inspection of code
- docker
- docker pipeline - to build and use docker containers from pipelines
- kubernetes
- kubernetes CLI
- kebernetes client api (just)
- kubernetes credentials (just)
click install

note: there are 2 diff things, 
1. sonarqube scanner - this is the tool thats gonna analyse and generate report
2. sonarqube server -  the portal where the reports will be published 


LETS CONFIGURE THE PLUGINS NOW
go to tools under manage jenkins

add names and installers for plugins.
jdk17, sonarqube-scanner, maven3, docker


Manage Jenkins - > System
Add sonarqube server, add url and select credentials.

LETS CREATE A PIPELINE
Click new items from side bar.
select pipeline , name it Boardgame
create pipeline

things to do: 
Go to manage - security -> Credentials. add new. (add them as global credentials)
- username and password : create an ID otherwise itll generate a long id which is hard to recognise.
- secret text : For sonarqube server credentials.(from sonarqube server -> administration - > security -> users -> tokens -> generate a token -> copy)


now lets configure it.
1. Check Discard old builds
    - Best practice to keep max 2 builds in my history, cuz i dont wanna occupy space

2. Pipeline
    - since we r gonna write from scratch, good to start with helloworld template.
    - you can generate commands from pipeline syntax. 

    1. Stage 1 : Git checkout
        geenrate command from pipeline sintax
        when cloning git we cannot use password since it was outdated, shud use token whih we created esrlier 
    2. Stage 2 : Compile
        sh "mvn compile"
    3. Stage 3 : Test
        sh "mvn test"
    4. Stage 4 : File System scan
        Note - we r gonna use Trivi, for this tool plugin is not available so shud install it in the jenkins server. 
    5. Stage 5 : SonarQube Analysis
    6. Stage 6 : Quality Gate (means some coditions that is there to assess the quality of sonarqube scan) to access that we need to create a webhook from the sonarqube server.
    7. Stage 7 : Build
        sh "mvn package"
    8. Stage 8 : Publish to Nexus (publishing artifacts to nexus)
        for this make sure u have added the repo urls in pom.xml of ur project. under <distributionManagement> . for repository and snapshot repo. which will publish artifacts in to nexus.

        then setup the Nexus credentials to access nexus from jenkins. manage jenkins-> managed files-> add new file-> global maven settings.xml. (which will be neede when publishing artifacts to nexus)

        sh "mvn deploy"   (shud be enclosed inside the command saying to use global settings.xml)

    9. stage 9 : Build and tag docker image







    - check vulnerabilities in the dependencies were using. (which is in pom.xml of project)
    - to find if there are any sensitive data that is breing stored in repo


 






Tools used 
- sonarqube - an opensource platform for continuous inspection of code


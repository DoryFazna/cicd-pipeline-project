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
- install kubectl (will be used from pipeline)
- install node-exporter (get the node exporter link from web)

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
- prometheus metrics
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



ADD NECESSARY CREDENTIALS IN JENKINS
Go to manage - security -> Credentials. add new. (add them as global credentials,create an ID otherwise itll generate a long id which is hard to recognise.)

- Git credential (username and password)
- Sonarqube server credentials. (secret text)(from sonarqube server -> administration - > security -> users -> tokens -> generate a token -> copy)
- docker cred (username pass)
- mail cred (username pass)
    - go to the gmail account -> manage account -> security -> App password.
    - provide app name as jenkins. copy that password, we r gonna use that to send mails.
- kubernetes cred which is a token (secret text) how to generate one? look below

RBAC -> Role BAse Access Control, u know what it is. which we r gonna do it to make our deployment make secure.
HOW TO DO THIS? 
1. Lets create service account in master node.
Go to master node 
  - create a file. vi svc.yaml, add user info
      apiVersion: v1
      kind: ServiceAccount
      metadata:
        name: jenkins
        namespace: webapps

  - create a namespace called webapps.
      kubectl create ns webapps

  - now run the command to create the service account.
      kubectl apply -f svc.yaml

2. Lets create the roles.
  - create a file role.yaml, and add the info.
  - run the apply command to create it just like how u did in previous step.

3. Now we need to bind the user to role.
  - create a file bind.yaml. add the content.
  - run apply.

4. inorder to make the jenkins access kubernetes server, we need to create token to authenticate.
how to do that? same as above..
  - create a file. sec.yaml. enter the details. make sure the service account name is jenkins.
  - when running it u can use the -namespace tag to denote the namespace. which is webapps in this case.
  kubectl apply -f sec.yaml -n webapps 

5. How to get the token now?? 
  kubectl describe secret mysecretname -n webapps
  (you will get the token there.. copy it and create a jenkins global credential named k8-cred. you know how to do it. and itll be ready to use in the pipeline script!)


Manage jenkins > Configure Mail notifiation
- find Extended Email notifiaction.
- SMTP server : smtp.gmail.com
- SMTP port : 465
- use SSL
- select mail credential which we created earlier.
- do the same for Email notification.
- Test and see under test configuration. 

-----------------------------------------------

NOW all set, LETS CREATE A PIPELINE
Click new items from side bar.
select pipeline , name it Boardgame
create pipeline


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
        - check vulnerabilities in the dependencies were using. (which is in pom.xml of project)
        - to find if there are any sensitive data that is breing stored in repo
    6. Stage 6 : Quality Gate (means some coditions that is there to assess the quality of sonarqube scan) to access that we need to create a webhook from the sonarqube server.
    7. Stage 7 : Build
        sh "mvn package"
    8. Stage 8 : Publish to Nexus (publishing artifacts to nexus)
        for this make sure u have added the repo urls in pom.xml of ur project. under <distributionManagement> . for repository and snapshot repo. which will publish artifacts in to nexus.

        then setup the Nexus credentials to access nexus from jenkins. manage jenkins-> managed files-> add new file-> global maven settings.xml. (which will be neede when publishing artifacts to nexus)

        sh "mvn deploy"   (shud be enclosed inside the command saying to use global settings.xml)

    9. stage 9 : Build and tag docker image
        - since its gonna be public no need mention registry url, 
        - you can add docker credentials thru pipeline syntax
        sh "docker build -t <username>/boardshack:latest ." 
        (dot in the end is the path u wanna add, im just gonns put it on root thats why dot.)

    10. stage 10 : Docker image scan 
        - using trivy.. enter command here

    11. Stage 11 : Push docker image
        sh "docker push <username>/boardshack:latest"

    12. Stage 12 : Deploy to Kubernetes
        - get the kubernetes server endpoint and cluster name
            1. visit the kubernetes root dir from the master node :  cd ~/.kube
            2. read the config file and fine the endpoint and cluster name: cat config 
        
        make sure u update the correct info , in deployment-service.yaml file of the board game project repo.
        - name
        - docker image name details
        - port (the app wher is gonna run in container), target port (the container port)
        - service type (loadbalancer)
            There are few more types : 
              1. Cluster IP : a service type used for internal communications (cannot access teh app from outside)
              2. loadbalancer : can access from outside
              3. node port : can access from outside but, not very useful now cuz it opens a port on worker node.
                The access of pods are gonna work according to the mentioned service type.

        Once ur done with specifying all details, write the script in pipeline
        sh "kubectl apply -f deployment-service.yaml"

    13. Stage 13 : Verify the deployment
        sh "kubectl get pods -n webapps"
        sh "kubectl get svc -n webapps"


    14. POST actions. 
          ex- Send email notification (u can attach trivy report)

DONE WITH PIPELINE!! 

RUN AND SEE.


MONITORING

- create a new instance : Monitor node.
- install necessary tools
    1. prometheus (follow steps on web)
      - run promethues. (./prometheus &) ampersand sign to run in bg
      - by default it runs on port 9090. so check it by accessing it.
    2. Grafana (follow steps on web)
      - start grafana. ( sudo /bin/systemctl start grafana-server)  
      - by default it runs on port 3000. so check it out. (default user,pass  admin,admin)
    3. Blackbox exporter (follow steps on web - in same page wher u downloaded prometheus)
      - start blackbox-exporter (./blackbox_exporter &)
      - default port 9115. check if its running.
- edit prometheus.yaml file and add necessary scrape configs.
    1. modify the blackbox exporter's hostname and port
    2. add new target url (boardgame)
- restart prometheus
    1. first kill the existing one 
        get the id -> pgrep prometheus
        kill it -> kill <id>
    2. run promethus now -> ./prometheus &
    3. visit the web and check it out if its working

- Add prometheus as a datasource in Grafana
    Grafana dashboard -> connections -> Data sources - > Add datasource -> prometheus -> Add prometheus server url.
    Save it
- Create new Dashboard in premetheus
    Click on plus icon -> import dashboard
    Choose a suitable dahsboard from the grafana.com/dashboards, copy the id and paste
    Select the datasource (in our case prometheus datasource one which we created)
    Click import and we are DONE!!


SYSTEM LEVEL MONITORING
- go to jenkins plugins and install prometheus metrics if u havnt(u cn configure it after tht in manage jenkins, but default configs are fine)
- go get the node-exporter from prometheus and install it in jenkins node if u havnt.
- run node-exporter 
- default port 9100, access it and check 
- Now again edit prometheus.yaml, add new targets to scrape. which are
    1. jenkins url
    2. node-exporter 
- restart prometheus - and u will see new probes
- now go to grafana and create new dashboard

TO DO : Currently we r using node-exporter for system level monitoring, blackbox-exporter for out webapp monitoring. Try and find how to use Node-exporter to monitor the website.


Tools used 
- sonarqube - an opensource platform for continuous inspection of code
- prometheus
- grafana
- blackbox-exporter - monitor website itself 
- node-exporter - monitor system level







# Deploying Viya4.0 (PSGEL255)
- #### [Course](https://eduvle.sas.com/course/view.php?id=1968) | [Hands-On](https://gitlab.sas.com/GEL/workshops/PSGEL255-deploying-viya-4.0.1-on-kubernetes)

---

##### [Chapter 01. Introduction](#Ch01)
##### [Chapter 02. Kubernetes and Containers Fundamentals](#Ch02)
##### [Chapter 03. Viya 4 Software Specifics](#Ch03)
##### [Chapter 04. Pre-Requisites](#Ch04)
##### [Chapter 05. Deployment Tools](#Ch05)
##### [Chapter 06. Deployment Steps](#Ch06)
##### [Chapter 07. Deployment Customizations](#Ch07)
##### [Chapter 08. Recommended Practices](#Ch08)
##### [Chapter 09. Validation](#Ch09)
##### [Chapter 10. Troubleshooting](#Ch10)
##### [Chapter 11. Azure AKS](#Ch11)

---

### Chapter 01. Introduction <a name='Ch01'></a>
**1.1 Workshop Introduction**
- Viya 4 has Cadence (Version) and Release
- Manual and Automated (Cheatcodes) processes
- Kubernetes 1.18 on Azure AKS

**1.2 Reference Material Index**
- **[Documentation in SAS Help Center (Test Environment)](http://pubsweb.na.sas.com/eds2/edittest/DraftDocHub.html) (The most important document!)**
- [Kubernetes standard icons](https://github.com/kubernetes/community/tree/master/icons)
- YAML: YAML Ain't Markup Language
- kubectl: Kubernetes CLI
- kustomize: Tool to generate Kubernetes manifests
- helm: Tool to deploy applications on Kubernetes easier
- rancher: Software to administratge Kubernetes easier
- SAS Viya Monitoring for Kubernetes:
  - prometheus / grafana: Software to monitor Kubernetes easier
  - kibana: Software to aggregae and search logs easier
- Visual Studio Code: Text editor that is recommended
- yq: Like jq, but for YAML query
- sed: Way to search and replace in Linux
- terraform: Infrastructure as Code tooling. Allows to spin IaaS resources up and down

**1.3 Lab Environment Information**
- [Viya 4 - Deployment Collection [GEL] Viya 4 - Deployment Collection](http://race.exnet.sas.com/Reservations?action=new&imageId=220997&imageKind=C&comment=C1%20-%20Viya%204%20Deployment%20-%205-Machine%20K8s%20Cluster&purpose=PST&sso=PSGEL255&schedtype=SchedTrainEDU&startDate=now&endDateLength=0&discardonterminate=y) 
- "sas-gelsandbox" Azure subscription
- Execute the following command on sasnode01:
  ```bash
  tail -f /opt/raceutils/logs/$(hostname).bootstrap.log
  ```
- Wait for the last line to say:
  ```bash
  03/16/20 18:15:38: Done running script /tmp/testclone/scripts/loop/GEL.99.TheEnd.sh
  ```
- Execute the following commands to see if Kubernetes is responding:
  ```bash
  kubectl cluster-info
  kubectl get nodes 
  ```
  
**1.4 Viya 4 - Highlights**
- Software is packaged and delivered as container images
  - YUM packages, RPMs, .msi file, etc. are no longer available
- Kubernetes 1.18+ for Viya 2020.1
- ***Viya 4 is supported on-cloud or on-prem***
  - Wherever you Kubernetes is
- Customers can deploy in their own Kubernetes OR
- Customers can have it hosted by SAS (on SAS Cloud) OR
- Customers can get SAS Remote Managed Service to control their Cloud
- **Viya 4 Cadences (Tracks, Version)**
  - ***Stable (Every Month)***
  - ***LTS (Every 6 Months) - Long Term Support (LTS)***
- Default install of Viya 2020 will not automatically keep itself updated. However, you can
  - Automate the process
  - Requires extra CI/CD tools to be used and configured by customer
  - Use the SAS Deployment Operator

**1.5 Deploying Viya 4 on Kubernetes - High-Level Introduction**

**Main steps for deployment from scratch:**
  1. Obtain Deployment Asseets
  2. Create/Adjust Deployment Files
  3. Build Deployment Manifest
  4. Apply Deployment Manifest

**1. Obtain Deployment Asseets**
  - Obtain it from the my.sas.com portal
  - It is a file with .tgz extension
  - Choose the latest

**2. Create/Adjust Deployment Files**
  - Create a file called "kustomization.yaml"
  - This is how you chooses:
    - SMP or MPP
    - Postgres in Pod or Postgres in Managed Cloud Database
    - HA or no HA
    - URLs seen by end-user
    - Name of Kubernetes Namespace in which to deploy

**3. Build Deployment Manifest**
  - Complies all the deployment files ito a single Manifest file
  - Default name of Manifest is "site.yaml"
  - Between 20,000 and 60,000 lines of YAML
  - Describes what your deployment is suppoesed to look like, by describing Kubernetes objects (Deployment, Statfulsets, Persistent Volumes, Ingresses, Services, Secrets, ConfigMaps, etc.)
  - Creatin of the manifest file does not deploy the software
 
**4. Apply Deployment Manifest**
- Applying the maifest instructs Kubernetes to create (or update) al the Kubernetes ojects described inthe manifest
- This will both "deploy" and "start" the software simultaneously

**What if deployment fails?**
- Small mistake (wrong ingress name, SMP instead of MPP)
  - Re-run through the steps (except downloading assets)
- Big mistake (wrong order, wrong version, wrong namespace, or method above does not work)
  - Delete the Namespace
  - Re-create the Namespace
  - Re-run though all the steps

### Chapter 02. Kubernetes and Containers Fundamentals <a name='Ch02'></a>
**2.1 Containers Basic Concepts**
- Container technology is not new (BSD Jails, Solaris Zones, AIX WPARS)
- [What are containers and why do you need them?](https://www.cio.com/article/2924995/what-are-containers-and-why-do-you-need-them.html)
- [Containers at Google](https://cloud.google.com/containers/)
- [What is Docker and why is it so darn popular?](https://www.zdnet.com/article/what-is-docker-and-why-is-it-so-darn-popular/)

**2.2 Containers Basic Concepts - Images vs Instances**
- **Container Image:** "Like MS Word document, .doc"
- **Container Instance:** "Printed version of MS Word document"
- From an image we can generate multiple instances. All instances are identical "at birth".
- **Image = Docker Image**
  - Immmutable file, that is essentially a snapshot of a container
- **Container = Docker Container = Container Instance**
  - Result of running and image
- **Docker Registry:** contains Docker Images

**2.3 Containers Basic Concepts - Docker**
- Docker Engine: builds and runs your containers
- Docker Client: sends commands to the Docker Daemon inside the DOcker Image
- Docker CE (Community Edition) can be used free of charge
- Docker EE (Enterprise Edition) is not free and comes with support
  - Basic, Standard and Advanced Tiers

**2.4 Containers Basic Concepts - Images**
- "Dockerfile" contains a set of instuctions on how to build the image:
  - Layers/Ingredients:
    - Parent/Base Image
    - Libraries and pre-reqs
    - Actual Application
    - Configuration files
  - Instruction on how to assemble the layers
- Commnad used to build a new image:
```bash
docker build -t myposi:v01 -f ./Dockerfile .
```
- You can always add your own layers on top of the other images
- Save an image to a .tar file
```bash
docker image save centos:7  > /tmp/mynewcentosimage.tar
```
- Load an image from a .tar file
```bash
docker image load < /tmp/mynewcentosimage.tar
```

**2.5 Containers Basic Concepts - VMs vs Containers**
- Containers are a more lightweight virtualization. It is OS-level virtualizaiton.
- Containers and VMs are only alike in that they both provide isolation.
  - [Containerization is not Virtualization](https://medium.com/on-docker/containerization-is-not-virtualization-c7a9b841518)
  - [Containers Are Not Lightweight VMs](https://www.linuxfoundation.org/blog/2017/05/containers-are-not-lightweight-vms/)
- HyperVisor (Type 2) - VirtualBox, VMWare
- HyperVisor (Type 1) - VMWare vSphere (OS+Hypervisor all in one)
- VMs can be used for Stateless or Stateful application
- Containers are perfect for Stateless application

**2.6 Containers Basic Concepts - Inside or outside**
- Containers are isolated froma each other and can use the same ports.
- By default, ports inside the container are only available from inside the container.
- You need t opublish the ports to make them available externally (to the host, to anywhere)
  On the docker host itself, there can only be one component/service on the port

**2.7 Containers Basic Concepts - Immutable Architecture**
- Immutable Architecture:
  - Unchanging over time or unable to be changed
  - This of it as VM where everything is read-only
  - Don't update. Replace.
  - If you successfully launched one environment from a set of images, you will always be able to launch another environment
- Why Immutable Architecture?
  - Preventing a server from changing is key to keeping it running:
    - no human error
    - no unintended consequences
    - no degradation over time
    - no "snowflake" server
  - If a server was to change (drift away from desired state), all you have to do is kill it a replace with a fresh one, built from the original image
  - Trace-ability, rollback
    - If a new image “breaks” the application, it’s easy to roll back to the previous version
    - Not so easy when changes were manual, and mis-documented


### Chapter 03. Viya 4 Software Specifics <a name='Ch03'></a>
### Chapter 04. Pre-Requisites <a name='Ch04'></a>
### Chapter 05. Deployment Tools <a name='Ch05'></a>
### Chapter 06. Deployment Steps <a name='Ch06'></a>
### Chapter 07. Deployment Customizations <a name='Ch07'></a>
### Chapter 08. Recommended Practices <a name='Ch08'></a>
### Chapter 09. Validation <a name='Ch09'></a>
### Chapter 10. Troubleshooting <a name='Ch10'></a>
### Chapter 11. Azure AKS <a name='Ch11'></a>




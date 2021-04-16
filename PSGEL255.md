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

**2.8 Containers Basic Concepts - State**
- If you should back it up ***regularly***, it’s **stateful**
- If you can back it up only ***once***, it’s **stateless**
- **Stateful applications** (such as DB): 
  - The information will be reused over time and is of value to one or more consumers
  - The state reflects interdependent transactions over time
  - Changes over time
  - Needs to be backed-up regularly
- **Stateless applications** (such as mouse click): 
  - Typically large number of easy to replicate transactions
  - The results from actions are not seen as valuable over time
  - Stateless actions can ultimately trigger a stateful application to save the output
  - May require some redundancy to provide high availability for an application
- Containers in general are great at doing Stateless
  - Static Website that gets updated infrequently
  - Microservices which provide short lived actions whose state does not need to be preserved
  - Short batch job that pulls data from DB, creates forecast, and stores results back in DB
- Containers for Stateful applications require storage
  - Without storage external to the container the immutability of the container is no longer
  - Containerizing stateful applications requires careful design and considerations
  - For stateful applications to be containerized, the state will be persisted outside the container:
    - in a database
    - in an object Store (e.g. AWS S3)
    - in a local folder on the docker host
    - in a shared folder (e.g. NFS)
    - in a durable storage location (e.g. AWS EBS)
  - If persisted data gets corrupted ... environment is corrupted...

**2.9 Containers Basic Concepts - Storage**
- **Docker Storage:**
  - Container File System
  - Bind Mount
  - Docker Volume
  - tmpfs
  - Shared FS

- **Container File System**
  - A container has its own file system (/)
    - That filesystem can only be stored "somewhere on the disk of the docker host". 
    - The filesystem is really a clone of what's in the docker image
  - A container's filesystem is gone as soon as the container is terminated
- Containers can NOT access each other's file system

- **Bind Mount**
  - Bind Mounts allow you to mount any folder from the host into the container
  ```bash
  docker container run -v /data/:/data_from_host/ -it centos:7 ls -al /data_from_host/
  ```
  - You can mount the same folders into multiple containers
  - Whatever change was made to the mounted folder will survive the death of the container(s)
  - Bind mounts can also be used to mount a single file 
    - (e.g: /etc/hosts:/etc/hosts)

- **Docker Volumes**

  - Docker volumes are similar to Bind Mounts, except they are managed by docker

  ```bash
  docker volume create my_data
  docker container run -it -v my_data:/app centos:7 bash
  ```
  - They survive the container's death, they don't have a pre-determined size
  - By default they are stored on the docker host in:
	```bash
	/var/lib/docker/volumes/
	```

  - Volumes, like bind mounts, can be attached to multiple containers simultaneously
 
 - **Docker tmpfs mount**
   - tmpfs allows you to mount the Docker hosts' memory as if it was disk:
   ```bash
   docker run -it --tmpfs /app centos:7  df -h | grep app
   ```
   - This can be great for security and things you would not want to be stored on the file system
   - This does not survive the death of the container
   - 2 containers cannot share a single tmpfs mount. 

- **NFS mounts**
  - You can either:
    - Mount an NFS share onto the docker host, and then bind mount it into the container. OR
    - You can create a Docker Volume of type NFS , and mount that into the container
    ```bash
    docker volume create --driver local \
    	--opt type=nfs \
    	--opt o=addr=server02,rw \
    	--opt device=:/shareme/dir02 \
    	my_nfs_share
    ```
    ```bash
    docker run -it -v my_nfs_share:/nfs centos:7 bash
    ```
**2.9 Containers Basic Concepts - Image Registry**
- Docker Registries hold Docker Images
  - Like a YUM mirror contains RPM packages
  - Like github contains code
- docker push: the action of uploading a copy of the local image up into a docker registry
- docker pull: the action of downloading a copy of an image from the registry onto the local machine

- Docker Registry ≠ Docker Repository
  - Registry: Holds images. Hands them out when asked nicely
  - Repository: a collection of related Docker images often with the same name but with different tags (versions) of the same application
  - [Difference between Docker registry and repository](https://stackoverflow.com/questions/34004076/difference-between-docker-registry-and-repository)

- Image ID is a unique id of an image
- Tag maps a descriptive user-given name to an image ID

- When do you need a registry?
  - Docker registry is:
    - required when containers run on Kubernetes
    - the customer is responsible for providing you with the registry information and how to connect to it
  - Docker registry is:
    - Optional when containers run on a single Docker Host
      (because the host that builds the image has the local version of the image)
    - If you need to use an image on multiple machines ... 
      you could    export   -    scp   -    import 
      OR you could push your image to a registry and tell your colleagues how to pull the image down 
      
**2.10 Kubernetes 101**
- Kubernetes Cluster:
  - Set of machines (computers) for running containerized applications that are called Nodes
- Kubernetes v1.18 cluster can have:
  - No more than 5,000 nodes
  - No more than 150,000 pods
  - No more than 300,000 containers
- Kubernetes Nodes:
  - Nodes can e a physical or virtual instance of a server
  - Nodes can vary in size
  - Nodes will host your applications, running as containers
  - Nodes can be independenctly started and stopped
- Containers
  - Instead of configuring a machine to host your application, you have your application wrapped into a container
  - The application is wrapped along with everything the application neeeds - OS, libraries, dependencies, etc. 
  - Containers are deployed on a machine that hosts the container engine which is in charge of actually running the containers
  - Containers technically don't belong to Kubernetes; they're just one of the tools of the trade
- Kubernetes Pods
  - A Pod is the smallest deployable compute object in Kubernetes
  - A Pod encapsulates an applications' container and related storage resources
  - A Pod can hold multipl containers, but usually contain one container
  - Pods are hosted on the Nodes, Kubernetes will determine which Node will host a Pod
  - A Pod is assigned a unique network identiy (IP address) which is independent of the node on which the pod is running
 
- Immutability and Declarative Deployment are conrnerstones of Kubernetes
  - Immutable architecture: unchanging over time or unable to be changed
  - Declarative Deploymnet: tell Kubernetes what the desired state should be (you declare it)
    - Kubernetes uses Deployments to manage these requirements

- Kubernetes Deployments
  - It is Kubernetes job to ensure that the desired state is achieved
  - That the required number of pod replicas are running

- Kubernetes Service
  - A Kubernetes Service is an abstraction which defines a logical set of Pods and a policy by which to access them
  - But why do I need a service?
    - Pods are transient. They start, they die, get restarted, they can be randomly assigned to a Kubernetes node
    - Pods don't move from node to node, they are replaced (stopped & restarted)
    - There may be more than one replica of a Pod
    - So we need a "static" way of accessing a Pod or group of Pods
    - The Service provides this mapping

- Kubernetes NodePort
  - A Kubernetes NodePort is a type of Publishing Service
  - For some parts of your application you may want to expose a Service onto an external IP address, you can use a NodePort to do this
  - A NodePort exposes the Service on each Node's IP at a static port

- Kubernetes Ingress & Ingress Controllers
  - Services support the communication from within the Cluster
  - An Ingress exposes HTTP and HTTPS routes from outside the Cluster to Services within the Cluster
  - Traffic routing is controlled by rules defined onthe Ingress resource
  - An Ingress Controller is responsible for fulfilling the Ingress
  - Commonly used Ingress Controllers include:
    - Traefix, NGINX, Istio, AWS ELB ..

- Kubernetes Namespace
  - Namespaces are a way of creating "virtual clusters within a single physical Kubernetes cluster
    - Namespaces provide a way to divide cluster resources between multiple applications (via resource quota)
  - Namespaces provide logical separation and a mechanism to logically group Kubernetes objects
  - Namespaces provide a useful mechanism to provide some level of multi-tenancy

- Kubernetes Volumes
  - The problem: On-disk files in a Container are ephemeral
  - So how do you make use of different storage types and persist data?
    - The Kubernetes Volume abstration allows data to persist and/or be shared between Containers in a Pod or shared across Pods
    - A kubernetes volume is just a directory (possibly with some data in it) which is accessible to the Container(s) in a Pod
    - Kubernetes support many types of volumes, and a Pod can use any number of them simultaneously
    - The lifetime of the volume depends on its type
    - The Volume types include (and more):
      - local, hostPath, emptyDir
      - nfs, persistentVolumeClaim

- Kubernetes Operators
  - Kubernetes operator pattern
    - Operators are software extensions to Kubernetes that make use of custom resources to manage applications and their components
  - Operators can be used to automate many tasks, including:
    - Deploying an application
    - Handling application upgrades
    - Performing admin tasks such as taking system backups
  - You can use an Operator to deploy the SAS Viya 4 software
    - The SAS Viya Deployment Operator provides and automated method for deploying and updating your SAS Viya deployment
    - The operator is not part of the SAS Viya deployment but is deployed in the SAS Viya namespace
    - Use of the SAS Viya Deployhment Operator is optional

**2.11 Kubernetes Basic Concepts - Installing**
- Managed Kubernetes Solutions
  - Many technology vendors provide access to hosted Kubernetes deployments/platforms and services to create Kubernetes platforms
  - [Kubernetes Documentation / Getting started](https://kubernetes.io/docs/setup/)
  - [Kubernetes Partners](https://kubernetes.io/partners/#kcsp)
- Install Kubectl
  - Most interactions with Kubernetes are done from the command line, [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) client tool
    - You can install kubectl on Linux, Mac, Windows
  - The kubectl client is useless unless it's properly configured:
  ```bash
  ~/.kube/config
  ```
  - Auto-completion for kubectl is a really good idea
  ```bash
  yum install bash-completion -y
  echo "source <(kubectl completion bash)" >> ~/.bashrc
  ```

**2.12 Kubernetes Basic Concepts - Namespaces**
- Exercise: 02 131 Learning about Namespaces
```bash
# List of available namespace
kubectl get ns

# The Quick way - Create a namespace
kubectl creat ns hare

# The Longer way - Creat a namespace
# Create manifest file
tee /tmp/turtle_namespace.yaml > /dev/null << EOF
---
apiVersion: v1
kind: Namespace
metadata:
  name: turtle
  labels:
    name: "my_turtle"
    owner: "sas.com"
EOF

# Apply the manifest
kubectl apply -f /tmp/turtle_namespace.yaml

# Describing the namespace
kubectl describe ns hare turtle

# Deleting the namespace
# The Fast way
kubectl delete ns hare

# The Slow way
kubectl delete -f /tmp/turtle_namespace.yaml
```
- A namespace has no size, unless a specific limit is set
- A namespace is not specific to any node
- Namespaces are mostly isolated from each other
  - By default, apps can communicate with each other across namespaces
  - Unless you configure specific network policies to disallow it
- Display what pods are in a namespace:
```bash
kubectl -n prod get pods
```
- Permanently change the default namespace:
```bash
kuectl config set-context --current --namespace=prod
kubectl get pods
```

**2.13 Kubernetes Basic Concepts - Management and CLI**
- To be efficient, you'll have to use the kubectl CLI
```bash
kubectl version --short
kubectl version

kubectl cluster-info
kubectl get nodes
kubectl get ns

kubectl create namespace mytest
kubectl get ns
kubectl delete ns mytest

kubectl get all -n d <tab>
kubectl <tab>
kubectl get <tab> <tab>

kubectl describe node intnode03
kubectl describe node nodes

# Look at Pods
kubectl -n kube-system get po
kubectl -n kube-system get po -o wide
kubectl -n kube-system get po -o json
kubectl get po --all-namespaces

kubectl -n kube-system describe po canal-546dh

# Manifest files
tee /tmp/sleep.yaml > /dev/null << EOF
---
apiVersion: v1
kind: Pod
metadata:
  name: busybox-sleep
spec:
  containers:
    - name: busybox
      image: busybox
      args:
      - sleep
      - "1000000"
EOF

kubectl apply -f /tmp/sleep.yaml
kubectl get po -o wide
kubectl exec -it busybox-sleep -- ps -ef
```

- TMUX is useful tool to keep watching activity or monitoring process
```bash
tmux <enter>
watch kubectl -n monitoring get po -o wide

# Devide screen
<ctl-b> then <">

# Exit from TMUX
exit <ctrl+c> exit

# Detach TMUX
<ctl-b> <d>
```

**2.13 Kubernetes Basic Concepts - Ingress**
- Ingress is Kubernetes Application Access
  - It is a way to access the application runing inside your Kubernetes cluster
- Kubernetes definion of Ingress:
  - An API object that manages external access to the services in a cluster, typically HTTP
  - Ingress can provide load balancing, SSL termination and name-based virtual hosting
- A batch job running on K8S needs no ingress
- Ingress != Ingress Controller
- Ingress is the definition of how a service is accessed
  - which DNS alias to use
  - send you to which service inside the cluster
- Ingress Controller is the piece of software that makes it happen:
  - NGINX, Traefik, Kong
- You can have more than one Ingress Controller in your cluster
- Publishing Services (Service Types)
  - ClusterIP: 
    - An IP for the service, internal to the cluster. Allows you to reach a service regardless of how many pods are behind it or where the pods are. 
    - However, no port re-mapping.
  - NodePort:
    - A port listens on every single node in the cluster.
  - LoadBalancer:
    - When using Cloud providers, generaes ELBs etc.
  - ExternalName:
    - Single IP added to the master?
- Hostname and Aliases
  - How many ways can you reach a given machine?
    - IP, Hostname, FQDN, DNS Alias, Alias in /etc/hosts
  - Some software does not care what you used to get to a machine
    - httpd
  - Some software will honor your first request but force you to use a specific name for the subsequent ones. 
    - SAS 9.4 mid-tier 
  - Some Software will analyze the name you used and direct you to a place based on the name you used
    - Ingress controller 
  
### Chapter 03. Viya 4 Software Specifics <a name='Ch03'></a>
### Chapter 04. Pre-Requisites <a name='Ch04'></a>
### Chapter 05. Deployment Tools <a name='Ch05'></a>
### Chapter 06. Deployment Steps <a name='Ch06'></a>
### Chapter 07. Deployment Customizations <a name='Ch07'></a>
### Chapter 08. Recommended Practices <a name='Ch08'></a>
### Chapter 09. Validation <a name='Ch09'></a>
### Chapter 10. Troubleshooting <a name='Ch10'></a>
### Chapter 11. Azure AKS <a name='Ch11'></a>




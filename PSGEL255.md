# Deploying Viya4.0 (PSGEL255)
- #### [Slides](https://eduvle.sas.com/course/view.php?id=1968) | [Hands-On](https://gitlab.sas.com/GEL/workshops/PSGEL255-deploying-viya-4.0.1-on-kubernetes)

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
  - ***LTS (Every 6 Months)***

### Chapter 02. Kubernetes and Containers Fundamentals <a name='Ch02'></a>
### Chapter 03. Viya 4 Software Specifics <a name='Ch03'></a>
### Chapter 04. Pre-Requisites <a name='Ch04'></a>
### Chapter 05. Deployment Tools <a name='Ch05'></a>
### Chapter 06. Deployment Steps <a name='Ch06'></a>
### Chapter 07. Deployment Customizations <a name='Ch07'></a>
### Chapter 08. Recommended Practices <a name='Ch08'></a>
### Chapter 09. Validation <a name='Ch09'></a>
### Chapter 10. Troubleshooting <a name='Ch10'></a>
### Chapter 11. Azure AKS <a name='Ch11'></a>




# PROJECT 7: DEVOPS TOOLING WEBSITE SOLUTION
In this project we will implement a tooling website solution which makes access to DevOps tools within a corporate infrastructure easily accessible.

This tooling solution would be in the form of a three tier architecture****


![image](https://github.com/ettebaDwop/dareyProject7/assets/7973831/78231a41-8542-45b1-87c2-deb64c2db73c)

### Prerequisites
In this project you will implement a solution that consists of following components:
Infrastructure: AWS
Webserver Linux: Red Hat Enterprise Linux 8
Database Server: Ubuntu 20.04 + MySQL
Storage Server: Red Hat Enterprise Linux 8 + NFS Server
Programming Language: PHP
Code Repository: GitHub 

* For Rhel 8 server use this ami: 
  RHEL-8.6.0_HVM-20220503-x86_64-2-Hourly2-GP2 (ami-035c5dc086849b5de)
  
### Implementation
#### Step 1: Network File System (NFS) Server Preparation
- From the diagram above we will create 5 AWS EC2 instances to serve as:
   * 3 Webservers
   * 1 NFS server and
   * 1 storage server for the Database






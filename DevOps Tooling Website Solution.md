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

![Screenshot (382)](https://github.com/ettebaDwop/dareyProject7/assets/7973831/88cd2309-2ad1-4434-ab22-7bf79dd31903)

- The next step is to create 3 EBS logical volumes (*lv-opt, lv-apps,* and *lv-logs*):

![Screenshot (385)](https://github.com/ettebaDwop/dareyProject7/assets/7973831/c26994db-0831-4b04-a343-e5257c08d733)
  
- Next we will create mount points for the volumes created above:
    Mount lv-apps on /mnt/apps – To be used by webservers
    Mount lv-logs on /mnt/logs – To be used by webserver logs
    Mount lv-opt on /mnt/opt – To be used by Jenkins server in Project 8

- Fire up Git Bash terminal to connect to our EC2 instances:

- attach volumes to our NFS server
 ![Screenshot (390)](https://github.com/ettebaDwop/dareyProject7/assets/7973831/afb44c21-1414-4a76-b93f-3ca432889614) 


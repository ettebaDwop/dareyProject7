# PROJECT 7: DEVOPS TOOLING WEBSITE SOLUTION
In this project we will implement a tooling website solution which makes access to DevOps tools within a corporate infrastructure easily accessible.

This tooling solution would be in the form of a three tier architecture****


![image](https://github.com/ettebaDwop/dareyProject7/assets/7973831/78231a41-8542-45b1-87c2-deb64c2db73c)

## Prerequisites
In this project you will implement a solution that consists of following components:
Infrastructure: AWS
Webserver Linux: Red Hat Enterprise Linux 8
Database Server: Ubuntu 20.04 + MySQL
Storage Server: Red Hat Enterprise Linux 8 + NFS Server
Programming Language: PHP
Code Repository: GitHub 

* For Rhel 8 server use this ami: 
  RHEL-8.6.0_HVM-20220503-x86_64-2-Hourly2-GP2 (ami-035c5dc086849b5de)
  
## Implementation

### Step 1: Network File System (NFS) Server Preparation
- From the diagram above we will create 5 AWS EC2 instances to serve as:
   * 3 Webservers
   * 1 NFS server and
   * 1 storage server for the Database

![Screenshot (382)](https://github.com/ettebaDwop/dareyProject7/assets/7973831/88cd2309-2ad1-4434-ab22-7bf79dd31903)

- The next step is to create 3 EBS logical volumes (*lv-opt, lv-apps,* and *lv-logs*):

![Screenshot (385)](https://github.com/ettebaDwop/dareyProject7/assets/7973831/c26994db-0831-4b04-a343-e5257c08d733)
  
- Next we will create mount points for the volumes created above:
  
    * Mount lv-apps on /mnt/apps – To be used by webservers
    * Mount lv-logs on /mnt/logs – To be used by webserver logs
    * Mount lv-opt on /mnt/opt – To be used by Jenkins server in Project 8

- Fire up Git Bash terminal to connect to our EC2 instances:
  
![Screenshot (388)](https://github.com/ettebaDwop/dareyProject7/assets/7973831/f6c2dfed-35ee-455d-8631-5c7c5c548347)

- Attach volumes to our NFS server
  
 ![Screenshot (390)](https://github.com/ettebaDwop/dareyProject7/assets/7973831/afb44c21-1414-4a76-b93f-3ca432889614) 

 - Next we create partitions on the 3 disks (xvdf1, xvdg1 and xvdh1): Run,
   
  ``` 
   sudo gdisk /dev/xvdf
   sudo gdisk /dev/xvdg
   sudo gdisk /dev/xvdh
   lsblk
```

![Screenshot (397)](https://github.com/ettebaDwop/dareyProject7/assets/7973831/d8b76a56-b5bd-4a6a-8995-51fc27e7fa19)

- Next we will install LVM2:
  
  `sudo yum install lvm2 -y`
  
- We check for available disks by running the command:

`sudo lvmdiskscan`

![Screenshot (400)](https://github.com/ettebaDwop/dareyProject7/assets/7973831/f3123064-b4ba-4173-9fa1-49146a8f36c8)

- Create Physical volumes:
  
  `sudo pvcreate  /dev/xvdf1/  /dev/xvdg1  /dev/xvdh1`
  
![Screenshot (402)](https://github.com/ettebaDwop/dareyProject7/assets/7973831/69b5792f-8744-4b47-8d25-205e8ec388f0)

`sudo pvs`



- Create a volume group called *webdata-vg*:

  ```
  sudo vgcreate webdata-vg  /dev/xvdf1/  /dev/xvdg1  /dev/xvdh1
  sudo vgs #to check volume group created
  ```

![Screenshot (408)](https://github.com/ettebaDwop/dareyProject7/assets/7973831/80f26b0c-b493-4966-82db-089f9ed7dffd)

#### Create 3 Logical Volumes

```
sudo lvcreate -n lv-apps -L 9G webdata-vg
sudo lvcreate -n lv-opt -L 9G webdata-vg
sudo lvcreate -n lv-logs -L 9G webdata-vg
`sudo lvs`        #to confirm creation of these volumes run 
```

![Screenshot (427)](https://github.com/ettebaDwop/dareyProject7/assets/7973831/941502d1-4d8d-4ae6-bc91-b2809dcf89f8)

![Screenshot (428)](https://github.com/ettebaDwop/dareyProject7/assets/7973831/219666dc-8bc2-4fa8-b6a1-9ad5fdb10b21)

#### Format disks as xfs
```
sudo mkfs -t xfs /dev/webdata-vg/lv-apps
sudo mkfs -t xfs /dev/webdata-vg/lv-opt
sudo mkfs -t xfs /dev/webdata-vg/lv-logs
```
#### Create mount points on /mnt directory for lv-apps and lv-logs

`sudo mkdir /mnt/apps  /mnt/logs  /mnt/opt`

![Screenshot (430)](https://github.com/ettebaDwop/dareyProject7/assets/7973831/51071392-4a85-40a6-917a-143f4fa97245)

##![Screenshot (429)](https://github.com/ettebaDwop/dareyProject7/assets/7973831/6eafa6a1-e9d3-4eff-93b4-0b5325e3af4a)

- Next mount
  
```
sudo mount /dev/webdata-vg/lv-apps /mnt/apps
sudo mount /dev/webdata-vg/lv-logs /mnt/logs
sudo mount /dev/webdata-vg/lv-opt /mnt/opt
```
![Screenshot (431)](https://github.com/ettebaDwop/dareyProject7/assets/7973831/966863b8-9358-44ba-8c4b-fb77bce7ac84)

#### Install and configure NFS Server

Run the following commands:

```
sudo yum -y update
sudo yum install nfs-utils -y
sudo systemctl start nfs-server.service
sudo systemctl enable nfs-server.service
sudo systemctl status nfs-server.service
```
![Screenshot (432)](https://github.com/ettebaDwop/dareyProject7/assets/7973831/7af481bc-1c2d-4fd8-97a3-4e958bdc7c5b)

Set up permission that will allow our Web servers to read, write and execute files on NFS:

```
sudo chown -R nobody: /mnt/apps
sudo chown -R nobody: /mnt/logs
sudo chown -R nobody: /mnt/opt

sudo chmod -R 777 /mnt/apps
sudo chmod -R 777 /mnt/logs
sudo chmod -R 777 /mnt/opt

#Restart NFS

`sudo systemctl restart nfs-server.service`

```
- To configure access to NFS for clients within the same subnet (example of Subnet CIDR – 172.31.32.0/20 ), run the commands:
```
sudo vi /etc/exports

/mnt/apps <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/logs <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/opt <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)

Esc + :wq!

sudo exportfs -arv  # Export mount points so webserver can see them
```

![Screenshot (438)](https://github.com/ettebaDwop/dareyProject7/assets/7973831/ecc21703-1871-4458-921a-a5ea1548ba79)

- Check which port is used by NFS and open it using Security Groups (add new Inbound Rule)
  
`rpcinfo -p | grep nfs`

![Screenshot (439)](https://github.com/ettebaDwop/dareyProject7/assets/7973831/2b6ede1b-5481-4a92-8da0-7846f1e38af6)

###*Important note: In order for NFS server to be accessible from your client, you must also open following ports: TCP 111, UDP 111, UDP 2049*

![Screenshot (442)](https://github.com/ettebaDwop/dareyProject7/assets/7973831/3f6dcd91-ffa6-4e11-b1d3-7ffa1d8f4660)


### Step 2 — Configure Database Server

To install and configure a MySQL DBMS to work with remote Web Server we will follow these steps:

- Install MySQL server
- Create a database and name it *tooling*
- Create a database user and name it *"webaccess"*
- Grant permission to webaccess user on tooling database to do anything only from the webservers subnet cidr

![Screenshot (433)](https://github.com/ettebaDwop/dareyProject7/assets/7973831/98992f5a-e809-40dc-b574-411fd2438e81)

![Screenshot (434)](https://github.com/ettebaDwop/dareyProject7/assets/7973831/981332a1-893a-443b-a55a-569650268bb7)

### Step 3 — Prepare the Web Servers

The steps involved in preparing the Web Servers would include:

1. Configuring NFS client (this step must be done on all three servers)
2. Deploying a Tooling application to our Web Servers into a shared NFS folder
3. Configuring the Web Servers to work with a single MySQL database

*Connect to the Webserver_1 (already created at the start of the project)

![Screenshot (467)](https://github.com/ettebaDwop/dareyProject7/assets/7973831/36bb97a5-92e9-4774-8462-fffb3e8fe81e)

#### Install NFS Client

```
sudo yum update
sudo yum install nfs-utils nfs4-acl-tools -y
```
*Mount /var/www/ and target the NFS server’s export for apps. Verify that NFS was mounted successfully by running *df -h*. Make sure that the changes will persist on Web Server after reboot:

```
sudo mkdir /var/www
sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/apps /var/www
lsblk
df -h
```
![Screenshot (465)](https://github.com/ettebaDwop/dareyProject7/assets/7973831/59db6046-06b1-4a7f-830b-39f7ac53ba6c)

- Configure the /etc/fstab file
  
`sudo vi /etc/fstab`

#### Add the following line: *<NFS-Server-Private-IP-Address>:/mnt/apps /var/www nfs defaults 0 0*

![Screenshot (470)](https://github.com/ettebaDwop/dareyProject7/assets/7973831/30cb64de-51a5-44cc-b05f-3e254f1167d9)

- Install Apache, PHP and Remi's Repository run the following commands:
  
  ```
  sudo yum install httpd -y
  sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
  sudo yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-9.rpm
  sudo dnf module reset php
  sudo dnf module enable php:remi-7.4
  sudo dnf install php php-opcache php-gd php-curl php-mysqlnd
  sudo systemctl start php-fpm
  sudo systemctl enable php-fpm
  setsebool -P httpd_execmem 1
  ```
### *Repeat steps 1-5 for Weserver_2 and Webserver_3* 

- Verify presence of Apache file on the Webserver /var/wwww file and the NFS Server /mnt/apps file
- 
![Screenshot (471)](https://github.com/ettebaDwop/dareyProject7/assets/7973831/9c045493-b096-4d72-87d6-b8d877c3c4ed)

- Locate log folder for Apache on WebServer
`sudo mount -t nfs -o rw,nosuid 172.31.43.1:/mnt/logs /var/log/httpd`

-Repeat fstab configuration here for the log folder 

`sudo vi /etc/fstab`

#### Add the following line: *<NFS-Server-Private-IP-Address>:/mnt/log /var/log/httpd nfs defaults 0 0*

- Fork the tooling source code from Darey.io Github Account to your Github account. (Learn how to fork a repo here)
  
  ![Screenshot (475)](https://github.com/ettebaDwop/dareyProject7/assets/7973831/4e9b4a7c-9a36-4e16-8d17-c8e20f1d4603)
  
- Deploy the tooling website’s code to the Webserver. Ensure that the html folder from the repository is deployed to /var/www/html
  `   `
- Update the website’s configuration to connect to the database (in /var/www/html/functions.php file). Apply tooling-db.sql script to your database using this command mysql -h <databse-private-ip> -u <db-username> -p <db-pasword> < tooling-db.sql
- 
![Screenshot (480)](https://github.com/ettebaDwop/dareyProject7/assets/7973831/782b99d5-fc23-4bbf-9d09-f9552cc2feaa)

![Screenshot (481)](https://github.com/ettebaDwop/dareyProject7/assets/7973831/64c9d905-f441-4b6d-b24a-da230af1a57d)

![Screenshot (483)](https://github.com/ettebaDwop/dareyProject7/assets/7973831/41c55654-824c-41c2-9160-7427a539e04b)
- In the Database Server run the MySql command to create a new admin user with username: myuser and password: password:
  
```
INSERT INTO ‘users’ (‘id’, ‘username’, ‘password’, ’email’, ‘user_type’, ‘status’) VALUES
-> (1, ‘myuser’, ‘5f4dcc3b5aa765d61d8327deb882cf99’, ‘user@mail.com’, ‘admin’, ‘1’);
```
 
`select * from users;`

![Screenshot (482)](https://github.com/ettebaDwop/dareyProject7/assets/7973831/1ed46a13-aefa-4ace-958d-b252295aed7e)



Open the website in your browser http://<Web-Server-Public-IP-Address-or-Public-DNS-Name>/index.php and make sure you can login into the website with myuser user.
![Screenshot (484)](https://github.com/ettebaDwop/dareyProject7/assets/7973831/57c921ff-4aac-43c2-9775-4c28b41c2fc7)
![Screenshot (479)](https://github.com/ettebaDwop/dareyProject7/assets/7973831/37812cd1-ae80-430c-b403-06dd89b86fa5)



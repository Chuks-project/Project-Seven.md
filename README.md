## DEVOPS TOOLING WEBSITE SOLUTION


In Project Six, you implemented a WordPress based solution that is ready to be filled with content and can be used as a full fledged website or blog. Moving further we will add some more value to our solutions that your DevOps team could utilize.

This project builds on the 3-tier architecture we saw earlier in Project six by adding a Network file System(NFS) to our system architecture(website). The NFS host the static files needed to server our site to the public while the database stores the dynamic data. This architecture makes our webserver stateless: meaning we can easily configure and scale our webserver up or down as need arises.

In this project 7 also, you will implement a solution that consists of following components:

1.Infrastructure: AWS
2.Webserver Linux: Red Hat Enterprise Linux 8
3.Database Server: Ubuntu 20.04 + MySQL
4.Storage Server: Red Hat Enterprise Linux 8 + NFS Server
5.Programming Language: PHP
6.Code Repository: GitHub

You shall be adopting the following steps towards implementing your Tooling Website Solution.

## STEP 1 – PREPARE NFS SERVER

- Spin up a new EC2 instance with RHEL Linux 8 Operating System.

- Based on your LVM experience from Project 6, Configure LVM on the Server.

- Instead of formating the disks as ext4 you will have to format them as xfs

- Ensure there are 3 Logical Volumes. lv-opt lv-apps, and lv-logs

- Create mount points on /mnt directory for the logical volumes as follow:
        
        `
        Mount lv-apps on /mnt/apps – To be used by webservers
        Mount lv-logs on /mnt/logs – To be used by webserver logs
        Mount lv-opt on /mnt/opt – To be used by Jenkins server in Project 8
       `
- Install NFS server, configure it to start on reboot and make sure it is up and running

      sudo yum -y update`
      sudo yum install nfs-utils -y
      sudo systemctl start nfs-server.service
      sudo systemctl enable nfs-server.service
      sudo systemctl status nfs-server.service`
      
- Export the mounts for webservers’ subnet cidr to connect as clients. For simplicity, you will install your all three Web Servers inside the same subnet, but in production set up you would probably want to separate each tier inside its own subnet for higher level of security. To check your subnet cidr – open your EC2 details in AWS web console and locate ‘Networking’ tab and open a Subnet link:

- Make sure we set up permission that will allow our Web servers to read, write and execute files on NFS:

                                   
       `sudo chown -R nobody: /mnt/apps
        sudo chown -R nobody: /mnt/logs
        sudo chown -R nobody: /mnt/opt
        `      
        sudo chmod -R 777 /mnt/apps
        sudo chmod -R 777 /mnt/log
        sudo chmod -R 777 /mnt/opt
`     
     `sudo systemctl restart nfs-server.service`
                                             
- Configure access to NFS for clients within the same subnet (example of Subnet CIDR – 172.31.32.0/20 ):

      `sudo vi /etc/exports`
      
- Configure access to NFS for clients within the same subnet (example of Subnet CIDR – 172.31.80.0/20 ):

      `sudo vi /etc/exports`
      
  ![sudnet id](https://user-images.githubusercontent.com/65022146/199224678-732d8a65-3109-4031-8d0d-0be300129474.png)

      
      
 - Check which port is used by NFS and open it using Security Groups (add new Inbound Rule):

       `rpcinfo -p | grep nfs`
       
      
      ![rcp info](https://user-images.githubusercontent.com/65022146/199214378-7fb9e69b-e0dd-43f5-a215-527156fcb8b3.png)
       
 - Important note: In order for NFS server to be accessible from your client, you must also open following ports: TCP 111, UDP 111, UDP 2049 as seen below:
 
![Inbound rules](https://user-images.githubusercontent.com/65022146/199215726-323afdaf-9d28-4920-8d13-622f292951ff.png)


### Step-2 Configure the Database Server
 
- Install MySQL server

- Create a database and name it tooling

- Create a database user and name it webaccess

- Grant permission to webaccess user on tooling database to do anything only from the webservers subnet cidr

- Flush Privileges

- Show Databases

- When the above commands are run successfully it will appear on your console as seen been:

![Database and user created](https://user-images.githubusercontent.com/65022146/199226231-a4c7c008-a859-4fe0-8fc1-64b08a42ace4.png)


### Step 3 — Prepare the Web Servers

#### We need to make sure that our Web Servers can serve the same content from shared storage solutions, in our case – NFS Server and MySQL database.

- Launch a new EC2 instance with RHEL 8 Operating System

-Install NFS client

`sudo yum install nfs-utils nfs4-acl-tools -y`

- Mount /var/www/ and target the NFS server’s export for apps

`                         
     sudo mkdir /var/www`
     
     sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/apps /var/www
`

- Verify that NFS was mounted successfully by running df -h. Make sure that the changes will persist on Web Server after reboot:

      `sudo vi /etc/fstab`

- Add the following line

     `<NFS-Server-Private-IP-Address>:/mnt/apps /var/www nfs defaults 0 0`


- Install Remi’s repository, Apache and PHP

     
 `   
     sudo yum install httpd -y

     sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

     sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm

     sudo dnf module reset php

     sudo dnf module enable php:remi-7.4

     sudo dnf install php php-opcache php-gd php-curl php-mysqlnd

     sudo systemctl start php-fpm

     sudo systemctl enable php-fpm

     setsebool -P httpd_execmem 1
`

- Repeat steps 1-5 for another 2 Web Servers.

- Verify that Apache files and directories are available on the Web Server in /var/www and also on the NFS server in /mnt/apps. If you see the same files – it means NFS is mounted correctly. You can try to create a new file touch test.txt from one server and check if the same file is accessible from other Web Servers.

- Locate the log folder for Apache on the Web Server and mount it to NFS server’s export for logs. Repeat step №4 to make sure the mount point will persist after reboot.

- Fork the tooling source code from Darey.io Github Account to your Github account. (Learn how to fork a repo here)

- Deploy the tooling website’s code to the Webserver. Ensure that the html folder from the repository is deployed to /var/www/html


#### In the images below, it can be confirmed that our Web Servers can serve the same content from shared storage solutions, in our case – NFS Server and MySQL database. The webservers 2 and 3 were able to receive and produce the message(Commands) from the Ist Webserver

### WEBSERVER 1

![ist wbser send](https://user-images.githubusercontent.com/65022146/199230725-27b714c7-ca1e-41fd-b569-24d8da0aeaa0.png)


WEBSERSERS 2 & 3

![2 RECE](https://user-images.githubusercontent.com/65022146/199231459-50065761-a144-4e24-a975-7f29556c7222.png)
![3 RECE](https://user-images.githubusercontent.com/65022146/199231462-486cd5bb-a113-450b-b75c-a9b9c1c82e55.png)


- Open TCP port 80 to allow access from the browser

- Disable SElinux using this command: `sudo setenforce 0`. To make it parmanent use the command below and set SELINUX=disabled

    `sudo vi /etc/sysconfig/selinux`

- Update the website’s configuration to connect to the database (in /var/www/html/functions.php file). Apply tooling-db.sql script to your database using this command       
   `mysql -h -u -p < tooling-db.sql`

- Create in MySQL a new admin user with username: myuser and password: password:
      
   `       
    INSERT INTO ‘users’ (‘id’, ‘username’, ‘password’, ’email’, ‘user_type’, ‘status’) VALUES
    -> (1, ‘myuser’, ‘5f4dcc3b5aa765d61d8327deb882cf99’, ‘user@mail.com’, ‘admin’, ‘1’);
    `
    
 - Open the website and input the public IP address in your browser
 
 - If everything is run successfully without error, you should be able to see a Tooling login page like the one below and make sure you can login into the websute with myuser user.


![wbserver1 logged in](https://user-images.githubusercontent.com/65022146/199234972-a5282574-72ee-4d4c-8759-51306a58ac64.png)

![wbserver logged in](https://user-images.githubusercontent.com/65022146/199235044-b944f30a-14d1-4fe8-9f00-63ff955c8de3.png)
![WEBSERVER2 LOGGED IN](https://user-images.githubusercontent.com/65022146/199235052-e5062f1c-ff45-4129-9562-f6b08b09d354.png)
![WEBSERVER3 LOGGED IN](https://user-images.githubusercontent.com/65022146/199235056-9c8a9dd3-7e29-4987-8cf9-a9e80fab898e.png)


 - 

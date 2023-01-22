# Project 8: LOAD BALANCER SOLUTION WITH APACHE

In this project one will implement a solution that consists of following components:

Infrastructure: AWS
Webservers Linux: Red Hat Enterprise Linux 8
Database Server: Ubuntu 20.04 + MySQL
Storage Server: Red Hat Enterprise Linux 8 + NFS Server
Load Balancer: Ubuntu Server 20.04
Programming Language: PHP
Code Repository: GitHub

Step 0:  Preparing prerequisites

- Sign in to AWS free tier account and create four new EC2 Instances of t2.nano family with RedHat OS, and one new EC2 Instance of Ubuntu 20.04
- Name the new instances thus;
- Server A name - "NFS server" (Red Hat Enterprise Linux 8 + NFS Server)
- Server B name - "webserver 1" (Red Hat Enterprise Linux 8)
- Server C name - "webserver 2" (Red Hat Enterprise Linux 8)
- Server D name - "Apache Load Balancer" (Ubuntu Server 20.04)
- Server C name - "database server" (Ubuntu 20.04 + MySQL)

Prepare NFS Server:

Spin up a new EC2 instance with RHEL Linux 8 Operating System and Configure LVM on the Server.

`lsblk`

![View attached volumes](image/lsblk_1.PNG)

Use gdisk utility to create a single partition on each of the 3 disks

`sudo gdisk /dev/xvdf`

![xvdf partition](image/xvdf.PNG)

`sudo gdisk /dev/xvdg`

![xvdg partition](image/xvdg.PNG)

`sudo gdisk /dev/xvdh`

![xvdh partition](image/xvdh.PNG)

Use `lsblk` utility to view the newly configured partition on each of the 3 disks.

![View newly configured partitions](image/lsblk_2.PNG)

Use pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM

`sudo pvcreate /dev/xvdf1`

`sudo pvcreate /dev/xvdg1`

`sudo pvcreate /dev/xvdh1`

![Create physical volumes](image/pvcreate.PNG)

Verify that your Physical volume has been created successfully by running

`sudo pvs`

![Physical volume successfully created](image/pvs.PNG)

Use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg
Verify that your VG has been created successfully by running

`sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1`

`sudo vgs`

![volume group successfully created](image/vgcreate.PNG)

Use lvcreate utility to create 3 logical volumes; lv-opt lv-apps, and lv-logs

`sudo lvcreate -n apps-lv -L 14G webdata-vg`

`sudo lvcreate -n logs-lv -L 14G webdata-vg`

`sudo lvcreate -n opt-lv -L 14G webdata-vg`

`sudo lvs`

`lsblk`

![Three logical volumes created](image/lvcreate_lvs_lsblk_3.PNG)

format the disks as "xfs"

`sudo mkfs -t xfs /dev/webdata-vg/apps-lv`

`sudo mkfs -t xfs /dev/webdata-vg/logs-lv`

`sudo mkfs -t xfs /dev/webdata-vg/opt-lv`

![xfs disk format](image/format_as_xfs.PNG)

Create "/var/www/html" directory to store website files

`sudo mkdir /mnt/apps`

`sudo mkdir /mnt/logs`

`sudo mkdir /mnt/opt`

![create mount points](image/create_mount_pts.PNG)

`sudo mount /dev/webdata-vg/apps-lv /mnt/apps`

`sudo mount /dev/webdata-vg/logs-lv /mnt/logs`

`sudo mount /dev/webdata-vg/opt-lv /mnt/opt`

![mount Logical Volumes](image/mount_LVs.PNG)

Install NFS server, configure it to start on reboot and make sure it is up and running

`sudo yum -y update`

![Update NFS server](image/update_NFS_Server.PNG)

`sudo yum install nfs-utils -y`

![Install nfs-utils](image/install_utils.PNG)

`sudo systemctl start nfs-server.service`

`sudo systemctl enable nfs-server.service`

`sudo systemctl status nfs-server.service`

![Restart and enable NFS Client](image/restart_enable_status_NFS.PNG)

`sudo vi /etc/exports`

`sudo exportfs -arv`

![Export the mounts](image/exportfs.PNG)

Edit inbound rules for NFS server

![Edit inbound rules NFS](image/edit_inbound_rules_NFS.PNG)

CONFIGURE THE DATABASE SERVER:

Install MySQL server

`sudo apt update`

![update database server](image/update_db_server.PNG)

`sudo apt install mysql-server -y`

![install mysql server](image/install_mysql.PNG)

Create a database and name it tooling

mysql> `create database tooling;`

mysql> `create user 'webaccess'@'172.31.16.0/20' identified by 'password';`

mysql> `grant all privileges on tooling.* to 'webaccess'@'172.31.16.0/20';`

mysql> `flush privileges;`

![Create Tooling database](image/Create_db_tooling_grant_permissions.PNG)

mysql> `show databases;`

![Tooling Database created](image/db_tooling_creation_confirmed.PNG)

Prepare the Web Servers:

Install NFS client

`sudo yum install nfs-utils nfs4-acl-tools -y`

![install_NFS_client_webserver-1](image/install_NFS_client_webserver-1.PNG)

![install_NFS_client_webserver-2](image/install_NFS_client_webserver-2.PNG)

Mount /var/www/ and target the NFS server’s export for apps

`sudo mkdir /var/www`

`sudo mount -t nfs -o rw,nosuid 172.31.18.30:/mnt/apps /var/www`

![Mount /var/www/ for webserver-1](image/mount_target_NFS_webserver-1.PNG)

![Mount /var/www/ for webserver-2](image/mount_target_NFS_webserver-2.PNG)

`df -h`

![dh-f_webserver-1](image/dh-f_webserver-1.PNG)

![dh-f_webserver-2](image/dh-f_webserver-2.PNG)

`sudo vi /etc/fstab`

![open etc fstab webserver-1](image/open_etc_fstab_webserver-1.PNG)

![open etc fstab webserver-2](image/open_etc_fstab_webserver-2.PNG)

Install Apache

`sudo yum install httpd -y`

![Install Apache](image/install_httpd_webserver-1.PNG)

![Install Apache](image/install_httpd_webserver-2.PNG)

`sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm`

![install Epel-release](image/install_https_webserver-1.PNG)

![install Epel-release](image/install_https_webserver-2.PNG)

`sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm`

![Install Remi’s repository](image/remirepo_webserver-1.PNG)

![Install Remi’s repository](image/remirepo_webserver-2.PNG)

`sudo dnf module reset php`

![dnf module reset php](image/module_rreset_enable_webserver-1.PNG)

![dnf module reset php](image/module_rreset_enable_webserver-2.PNG)

`sudo dnf install php php-opcache php-gd php-curl php-mysqlnd`

![Install Opache](image/install_opache_webserver-1.PNG)

![Install Opache](image/install_opache_webserver-2.PNG)

`sudo systemctl start php-fpm`

`sudo systemctl enable php-fpm`

`sudo setsebool -P httpd_execmem 1`

![Restart php-fpm](image/restart_enable_setsebool_webserver-1.PNG)

![Restart php-fpm](image/restart_enable_setsebool_webserver-2.PNG)

`sudo mount -t nfs -o rw,nosuid 172.31.18.30:/mnt/logs /var/log/httpd`

![mount for logs WBS-1](image/mount_for_logs_WBS-1.PNG)

![mount for logs WBS-2](image/mount_for_logs_WBS-2.PNG)

`sudo vi /etc/fstab`

![Update etc/fstab/ webserver-1](image/update_etc_fstab_webserver-1.PNG)

![[Update etc/fstab/ webserver-2](image/update_etc_fstab_webserver-2.PNG)

Install git on Webservers

`sudo yum install git`

![Install git on Webserver-1](image/Install_git_wbs-1.PNG)

![Install git on Webserver-2](image/Install_git_wbs-2.PNG)

`git init`

`git clone https://github.com/darey-io/tooling.git`

![Fork Tooling Repo into webserver-1](image/repo_forked_wbs-1.PNG)

![Fork Tooling Repo into webserver-2](image/repo_forked_wbs-2.PNG)

Disable Selinux

`sudo vi /etc/sysconfig/selinux`

![open selinux file](image/open_selinux_file_WBS-2.PNG)

![Disable Selinux](image/disable_selinux_wbs-1.PNG)

![Disable Selinux](image/disable_selinux_wbs-2.PNG)

`sudo systemctl start httpd`

`sudo systemctl status httpd`

`sudo yum install mysql`

![Install mysql client](image/Install_mysql_client_wbs-1.PNG)

Step 1: Configure Apache As A Load Balancer

Create an Ubuntu Server 20.04 EC2 instance and name it Project-8-apache-lb, so your EC2 list will look like this

![Launched instances](image/EC2_instance-Apache-LB.PNG)

Open TCP port 80 on Project-8-apache-lb by creating an Inbound Rule in Security Group

Install Apache Load Balancer on Project-8-apache-lb server and configure it to point traffic coming to LB to both Web Servers:

Install apache2

`sudo apt update`

![Update Apache Load Balancer](image/update_Apache_LB.PNG)

`sudo apt install apache2 -y`

![Install Apache](image/install_Apache_LB.PNG)

`sudo apt-get install libxml2-dev`

![install_limbxml2-dev](image/install_limbxml2-dev_LB.PNG)

Enable following modules:

`sudo a2enmod rewrite`

`sudo a2enmod proxy`

`sudo a2enmod proxy_balancer`

`sudo a2enmod proxy_http`

`sudo a2enmod headers`

`sudo a2enmod lbmethod_bytraffic`

![Enable Apache dependencies](image/ennable_apache_dependencies_LB.PNG)

Restart apache2 service:

`sudo systemctl restart apache2`

![Restart Apache](image/restart_apache2_LB.PNG)

Make sure apache2 is up and running

`sudo systemctl status apache2`

![apache2 status](image/Apache_status_check_LB.PNG)

Configure load balancing

`sudo vi /etc/apache2/sites-available/000-default.conf`

![Configure load balancing](image/Updating_balancer_members_LB.PNG)

Restart apache server

`sudo systemctl restart apache2`

![Restart Apache server](image/restart_apache2_LB_2.PNG)

Verify that our configuration works – try to access your LB’s public IP address or Public DNS name from your browser:

`http://172.31.32.24/index.php`

If in the Project-7 you mounted /var/log/httpd/ from your Web Servers to the NFS server – unmount them and make sure that each Web Server has its own log directory.

Open two ssh/Putty consoles for both Web Servers and run following command:

`sudo tail -f /var/log/httpd/access_log`

Refresh your browser page

`http://172.31.32.24/index.php` several times and make sure that both servers receive HTTP GET requests from your LB – new records must appear in each server’s log file. The number of requests to each server will be approximately the same since we set loadfactor to the same value for both servers – it means that traffic will be distributed evenly between them.

![Webserver-1 logs](image/clients-logs-from-Webserver-1.PNG)

![Webserver-2 logs](image/clients-logs-from-Webserver-2.PNG)

Optional Step – Configure Local DNS Names Resolution:

Configure IP address to domain name mapping for our LB using the "/etc/hosts" file.

`sudo vi /etc/hosts`

![Add the arbitrary names for both of your Web Servers](image/Configure-Local-DNS-with-Arbitary-names.PNG)

Now you can update your LB config file with those names instead of IP addresses.

![update LB config file with domain names](image/Updated-LB-Config-file.PNG)

curl your Web Servers from LB locally

`curl http://Web1` or `curl http://Web2`

![Accessibility confirmed](image/HTML-of-tooling-page.PNG)

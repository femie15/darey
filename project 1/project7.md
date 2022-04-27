Spin-up 4 Redhat servers, 3 for web servers and the other for NFS server then spin up another Ubuntu server for database server, note the availability zones of the servers.

![apache](https://github.com/femie15/darey/blob/main/project%201/project7/1-instances.PNG)

Goto volumes and create 3 volume instances and attach them to the NFS-server.

then we can do `lsblk` on our terminal to view the attached volumes and `df -h` to view all mounts and free space on the server.

use command `gdisk /dev/xvdf` to initiate the partition process. it gives a prompt and we type `n` for new partition 

we then respond `1` for just a single partition (we can have more that 1, based on preference).

we then press enter twice to use up the whole space available

the current type of the system is usually "linux filesystem" we need to change to logical volumes by selecting (typing) `8e00` 
and then `p` to view what we have configured so far.

we then use `w` to write and confirm the command with a yes `Y`,

at this stage, it should show us that the operation is successfull.

So, we repeat the process for "xvdg and xvdh"

we can then run `lsblk` to view our volumes, each should now carry the number of partitions configured.

![apache](https://github.com/femie15/darey/blob/main/project%201/project6/4-xvdf.PNG)

![apache](https://github.com/femie15/darey/blob/main/project%201/project6/5-fgh.PNG)

we now install "LVM2" `yum install lvm2 -y`  (note, the -y flag is to auto respond yes to all promted questions)

to confirm that it was successfully installed, we can perform `which lvm` and see it in the installed directory

![apache](https://github.com/femie15/darey/blob/main/project%201/project6/6-install-yum.PNG)

run `lvmdiskscan` to see the partitions.

### Create physical volumes

we use the command `pvcreate /dev/xvdf1 /dev/xvdg1 /dev/xvdh1` to create physical volumes and use `pvs` to check the physical volumes

### Volume group

we use the volume group to put the physical volume resource into a single group from which a logical volume can then be created

run `vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1` to create a volume group called "webdata-vg" (in the future we can use vgextend to add more volume to the group)

we can do `vgs` to view the volume group (it should show us the sum of all the physical volumes we added)

![apache](https://github.com/femie15/darey/blob/main/project%201/project6/7-volume-group.PNG)

### Logical volume

This helps us to manage our volume easily without any downtime.

We use lvcreate utility to create 2 logical volumes. apps-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size

`lvcreate -n lv-apps -L 9G webdata-vg` for the application volume and

`lvcreate -n lv-logs -L 9G webdata-vg` for the logs, we can do `lvs` to see the logical volumes

`lvcreate -n lv-opt -L 9G webdata-vg` for the opt, we can do `lvs` to see the logical volumes

if we exhust the volume, we can create an additional physical volume, add it to our volume group by `vgextend` and add it to our logical volume by `lvextend` .

![apache](https://github.com/femie15/darey/blob/main/project%201/project7/2-plvg.PNG)

## View Setup

use `vgdisplay -v` you get the output below

![apache](https://github.com/femie15/darey/blob/main/project%201/project7/3-vgdisplay.PNG)


### File system

Use mkfs.ext4 to format the logical volumes with ext4 filesystem

`mkfs -t xfs /dev/webdata-vg/lv-apps`

`mkfs -t xfs /dev/webdata-vg/lv-logs`
 
`mkfs -t xfs /dev/webdata-vg/lv-opt`


![apache](https://github.com/femie15/darey/blob/main/project%201/project6/11-filesystem.PNG)

### create mount points for apps, logs and opt.

Create directory to store files, (it's good to check if the directory exists first before creating.

`mkdir -p /mnt/apps` (the `-p` flag creates the directories that are not currently existing)

`mkdir -p /mnt/logs`

`mkdir -p /mnt/opt`

### mount volumes

Check if the directories created are empty (mounting will automatically delete existing content in the directory)
then we can mount in the empty directory 

'mount /dev/webdata-vg/lv-apps /mnt/apps`

'mount /dev/webdata-vg/lv-logs /mnt/logs`

'mount /dev/webdata-vg/lv-opt /mnt/opt`


### Install NFS server, configure it to start on reboot and make sure it is u and running
`sudo yum -y update`  (This process takes time, so we can go and run the database process and ccome back to it.)
`sudo yum install nfs-utils -y` 
`sudo systemctl start nfs-server.service` 
`sudo systemctl enable nfs-server.service` 
`sudo systemctl status nfs-server.service` 

### Make sure we set up permission that will allow our Web servers to read, write and execute files on NFS:

`sudo chown -R nobody: /mnt/apps` 
`sudo chown -R nobody: /mnt/logs` 
`sudo chown -R nobody: /mnt/opt`

`sudo chmod -R 777 /mnt/apps` 
`sudo chmod -R 777 /mnt/logs` 
`sudo chmod -R 777 /mnt/opt` 

`sudo systemctl restart nfs-server.service` 

### Configure access to NFS for clients within the same subnet (example of Subnet CIDR – 172.31.32.0/20 ):

`vi /etc/exports`

and insert the following...

/mnt/apps <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/logs <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/opt <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)

on ketboard do "Esc + :wq!"

`exportfs -arv`

Check which port is used by NFS and open it using Security Groups (add new Inbound Rule)

`rpcinfo -p | grep nfs`

Important note: In order for NFS server to be accessible from your client, you must also open following ports: TCP 111, UDP 111, UDP 2049 and TCP 2049

![apache](https://github.com/femie15/darey/blob/main/project%201/project7/5-nfs%20ports.PNG)

![apache](https://github.com/femie15/darey/blob/main/project%201/project7/6-sec%20group.PNG)

# PART 2

## DB server

`apt update` to update the server

`apt install mysql-server -y`

`mysql` 

Create a database called tooling `create database tooling;`

create user called webaccess ensure we grant permission to webaccess user on tooling database to do anything only from the webservers subnet cidr (This can be found in aws instance > networking > subnetID > IPV4 CIDR)

`create user 'webaccess'@'172.31.0.0/20' identified by 'password';`

Grant privileges 
`grant all privileges on tooling.* to 'webaccess'@'172.31.0.0/20';`

To view the users we run `select user, host from mysql.user;`

![apache](https://github.com/femie15/darey/blob/main/project%201/project7/4-mysql%20users.PNG)

# Prepare the Web Servers

This approach will make our Web Servers stateless, which means we will be able to add new ones or remove them whenever we need, and the integrity of the data (in the database and on NFS) will be preserved.

Install NFS client on the 3web servers and repeat the processes below

`yum install nfs-utils nfs4-acl-tools -y`

Mount /var/www/ and target the NFS server’s export for apps

`mkdir /var/www`

`mount -t nfs -o rw,nosuid 172.31.6.219:/mnt/apps /var/www` "<NFS-Server-Private-IP-Address>"
 
Verify that NFS was mounted successfully by running `df -h`. 

when we add a file from one web server `touch /var/www/test.md`, it appears iin the NFS server and can be accessed from the other 2web servers as well using `ls /var/www/` we see the test.md file

![apache](https://github.com/femie15/darey/blob/main/project%201/project7/7%20-%20same%20file%20accessed%20across%20web%20servers.PNG)

Make sure that the changes will persist on Web Server after reboot:

 `vi /etc/fstab`
 
add following line

"172.31.6.219:/mnt/apps /var/www nfs defaults 0 0"

do these for one of the web servers

### Install Remi’s repository, Apache and PHP

`yum install httpd -y` 

`dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm` 

` dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm` 

` dnf module reset php` 

` dnf module enable php:remi-7.4` 

` dnf install php php-opcache php-gd php-curl php-mysqlnd` 

` systemctl start php-fpm` 

` systemctl enable php-fpm` 

`setsebool -P httpd_execmem 1` 

Repeat steps for all Web Servers.

Locate the log folder for Apache on the Web Server and mount it to NFS server’s export for logs. Repeat the "vi /etc/fstab" step to make sure the mount point will persist after reboot.

mount the web server log directory to the NFS server log directory

`mount -t nfs -o rw,nosuid 172.31.6.219:/mnt/logs /var/log/httpd`

Make sure that the changes will persist on Web Server after reboot:

 `vi /etc/fstab`
 
add following line

"172.31.6.219:/mnt/logs /var/log/httpd nfs defaults 0 0" this is the NFS internal IP Address.
and repeat the process for all web server.

run `yum install git` and `git init` for the web servers and fork a repository such as "https://github.com/darey-io/tooling.git" into it
Deploy the tooling website’s code to the Webserver.

`git clone https://github.com/darey-io/tooling.git` and then ls into the directory to see the html folder.

 Ensure that the html folder from the repository is deployed to /var/www/html
 
 `ls  tooling` then do `cp -R html/. /var/www/html` to copy the contents (-R means recurrsive)
 
 open TCP port 80 on the Web Servers.
 
 in the root directory of the web server
 
 disable SELinux by running `setenforce 0`
 
 To make this change permanent – open following config file 
 
 `vi /etc/sysconfig/selinux` and set "SELINUX=disabled" then restrt httpd with 
 
 `systemctl start httpd`
 
 Try to now access the web app via the public IP address. 
 
 If it's not loading try to change 
 
 1. "https" to "http"
 2. check permissions to your "/var/www/html" folder
 
 ### update db access
 
Update the website’s configuration to connect to the database (in /var/www/html/functions.php file). `vi /var/www/html/functions.php`

change the configuration from 

to "$db = mysqli_connect('172.31.13.186', 'webaccess', 'password', 'tooling');"

### Mysql client

on the 3 webservers install mysql client `yum install mysql -y`

then configure the database server security group to allow access to the NFS server Subnet CIDR

![apache](https://github.com/femie15/darey/blob/main/project%201/project7/8-dbSecGrp.PNG)

in the mysql server goto `vi /etc/mysql/mysql.conf.d/mysqld.cnf`

![apache](https://github.com/femie15/darey/blob/main/project%201/project7/9-mysql.cnf.PNG)

then restart mysql `systemctl restart mysql`

and edit the "bind-address" to "0.0.0.0" and "mysqlx-bind-address" to "0.0.0.0"

Goto the web server and apply tooling-db.sql (from the tooling folder forked from github) script to your database using this command in 

`cd tooling`

`mysql -h <databse-private-ip> -u <db-username> -p <db-name> < tooling-db.sql`

eg `mysql -h 172.31.13.186 -u webaccess -p tooling < tooling-db.sql` 

we can now check the db server if the sql script was successful.

`mysql` `show databases;` `use tooling;` `select * from users;`

also run 

INSERT INTO `users` (`id`, `username`, `password`, `email`, `user_type`, `status`) VALUES (2, "myuser", "5f4dcc3b5aa765d61d8327deb882cf99", "user@mail.com", "admin", "1");
note: the encrypted password is "password"

![apache](https://github.com/femie15/darey/blob/main/project%201/project7/10-dbData.PNG)

Open the website in your browser http://<Web-Server-Public-IP-Address-or-Public-DNS-Name>/index.php and make sure you can login into the websute with myuser user.

if the default redhat landing page is still showing, then we need to rename the file below
 
 `mv /etc/httpd/conf.d/welcome.conf /etc/httpd/conf.d/welcome.conf_rename` and then restart the web server `systemctl restart httpd`
 
# CONGRATULATIONS

 ![apache](https://github.com/femie15/darey/blob/main/project%201/project7/11-done.PNG)

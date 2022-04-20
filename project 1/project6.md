# WEB SOLUTION WITH WORDPRESS

Spin-up 2 server, one for wordpress web server and the other for MySQL database server, note the availability zones of the servers.

Goto volumes and create 3 volume instances and attach them to the web-server.

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

we now install "LVM2" `yum install lvm2 -y`  (note, the -y flag is to auto respond yes to all promted questions)

to confirm that it was successfully installed, we can perform `which lvm` and see it in the installed directory

### Create physical volumes

we use the command `pvcreate /dev/xvdf1 /dev/xvdg1 /dev/xvdh1` to create physical volumes and use `pvs` to check the physical volumes

### Volume group

we use the volume group to put the physical volume resource into a single group from which a logical volume can then be created

run `vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1` to create a volume group called "webdata-vg" (in the future we can use vgextend to add more volume to the group)

we can do `vgs` to view the volume group (it should show us the sum of all the physical volumes we added)

### Logical volume

This helps us to manage our volume easily without any downtime.

We use lvcreate utility to create 2 logical volumes. apps-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size

`lvcreate -n apps-lv -L 14G webdata-vg` for the application volume and

`lvcreate -n logs-lv -L 14G webdata-vg` for the logs, we can do `lvs` to see the logical volumes

if we exhust the volume, we can create an additional physical volume, add it to our volume group by `vgextend` and add it to our logical volume by `lvextend` .

## View Setup

use `vgdisplay -v` you get the output below

Use mkfs.ext4 to format the logical volumes with ext4 filesystem

`mkfs -t ext4 /dev/webdata-vg/apps-lv` 
`mkfs -t ext4 /dev/webdata-vg/logs-lv` OR 

`mkfs.ext4 /dev/webdata-vg/apps-lv` and `mkfs.ext4 /dev/webdata-vg/logs-lv`

### create directories for web files

Create /var/www/html directory to store website files, (it's good to check if the directory exists first before creating.

`mkdir -p /var/www/html` (the `-p` flag creates the directories that are not currently existing)

### Logs directory

Create /home/recovery/logs to store backup of log data

`mkdir -p /home/recovery/logs`

### mount volumes

Check if the directory "/var/www/html" is empty (mounting will automatically delete existing content in the directory)

then we can mount in the empty directory ` mount /dev/webdata-vg/apps-lv /var/www/html`

For logs, if we check the content, we notice it's not empty `ls /var/log`

We then need to backup the content in to the recovery placeholder using "rsync"

`rsync -av /var/log/. /home/recovery/logs/`

we can then mount the volume 
`mount /dev/webdata-vg/logs-lv /var/log`

at this point the contents in "/var/log" is already deleted, we then need to restore it back. 
`rsync -av /home/recovery/logs/. /var/log`. we can do `df -h` to view the mounted volume

We can now update /etc/fstab file so that the mount configuration will persist after restart of the server.
The UUID of the device will be used to update the /etc/fstab file

`UUID=5a75cdef-44e3-4222-8b6a-45139cb31eee /var/www/html ext4 defaults 0 0` 
`UUID=d7513fc9-56a5-4c28-adaf-89b543ceed24 /var/log ext4 defaults 0 0`

### Test the configuration and reload the daemon

`mount -a` to test and 
`systemctl daemon-reload` to reload

we can verify setup by running `df -h`

## DB server

Repeat the same steps as for the Web Server, but instead of apps-lv create db-lv and mount it to /db directory instead of /var/www/html/

# Install WordPress on your Web Server EC2

Update the repository `yum -y update`

Install wget, Apache and it’s dependencies `sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json`

Start Apache

`systemctl enable httpd` 
`systemctl start httpd`

To install PHP and it’s depemdencies

`yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

 yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
 
 yum module list php
 
 yum module reset php
 
 yum module enable php:remi-7.4
 
 yum install php php-opcache php-gd php-curl php-mysqlnd
 
 systemctl start php-fpm
 
 systemctl enable php-fpm
 
 setsebool -P httpd_execmem 1`


Restart Apache

`systemctl restart httpd`

Download wordpress and copy wordpress to var/www/html

  `mkdir wordpress
  
  cd   wordpress
  
  wget http://wordpress.org/latest.tar.gz
  
  tar xzvf latest.tar.gz
  
  rm -rf latest.tar.gz
  
  cp wordpress/wp-config-sample.php wordpress/wp-config.php
  
  cp -R wordpress /var/www/html/`
  
Configure SELinux Policies

`chown -R apache:apache /var/www/html/wordpress
 
 chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
 
 setsebool -P httpd_can_network_connect=1`








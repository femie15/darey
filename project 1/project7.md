Spin-up 2 server, one for wordpress web server and the other for MySQL database server, note the availability zones of the servers.

![apache](https://github.com/femie15/darey/blob/main/project%201/project6/1-servers.PNG)

Goto volumes and create 3 volume instances and attach them to the web-server.

![apache](https://github.com/femie15/darey/blob/main/project%201/project6/2-setup-volume.PNG)

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

![apache](https://github.com/femie15/darey/blob/main/project%201/project6/3-volumes.PNG)

![apache](https://github.com/femie15/darey/blob/main/project%201/project6/4-xvdf.PNG)

![apache](https://github.com/femie15/darey/blob/main/project%201/project6/5-fgh.PNG)

we now install "LVM2" `yum install lvm2 -y`  (note, the -y flag is to auto respond yes to all promted questions)

to confirm that it was successfully installed, we can perform `which lvm` and see it in the installed directory

![apache](https://github.com/femie15/darey/blob/main/project%201/project6/6-install-yum.PNG)

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

`lvcreate -n apps-lv -L 14G webdata-vg` for the application volume and

`lvcreate -n logs-lv -L 14G webdata-vg` for the logs, we can do `lvs` to see the logical volumes

if we exhust the volume, we can create an additional physical volume, add it to our volume group by `vgextend` and add it to our logical volume by `lvextend` .

![apache](https://github.com/femie15/darey/blob/main/project%201/project6/8-logical-volume.PNG)

## View Setup

use `vgdisplay -v` you get the output below

![apache](https://github.com/femie15/darey/blob/main/project%201/project6/9-phy-vgrp-logicvol.PNG)

![apache](https://github.com/femie15/darey/blob/main/project%201/project6/10-vgdisplay.PNG)

### File system

Use mkfs.ext4 to format the logical volumes with ext4 filesystem

`mkfs -t ext4 /dev/webdata-vg/apps-lv` 
`mkfs -t ext4 /dev/webdata-vg/logs-lv` OR 

`mkfs.ext4 /dev/webdata-vg/apps-lv` and `mkfs.ext4 /dev/webdata-vg/logs-lv`


![apache](https://github.com/femie15/darey/blob/main/project%201/project6/11-filesystem.PNG)

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

![apache](https://github.com/femie15/darey/blob/main/project%201/project6/12-mount.PNG)

at this point the contents in "/var/log" is already deleted, we then need to restore it back. 
`rsync -av /home/recovery/logs/. /var/log`. we can do `df -h` to view the mounted volume

### fstab
We can now update /etc/fstab file so that the mount configuration will persist after restart of the server.
The UUID of the device will be used to update the /etc/fstab file

`UUID=5a75cdef-44e3-4222-8b6a-45139cb31eee /var/www/html ext4 defaults 0 0` 
`UUID=d7513fc9-56a5-4c28-adaf-89b543ceed24 /var/log ext4 defaults 0 0`

![apache](https://github.com/femie15/darey/blob/main/project%201/project6/13-fstab.PNG)

### Test the configuration and reload the daemon

`mount -a` to test and 
`systemctl daemon-reload` to reload

we can verify setup by running `df -h`


## DB server

Repeat the same steps as for the Web Server, but instead of apps-lv create db-lv and mount it to /db directory instead of /var/www/html/

![apache](https://github.com/femie15/darey/blob/main/project%201/project6/15-fstab-db.PNG)

# Install WordPress on your Web Server EC2

Update the repository `yum -y update` (for both web and database server: note this takes some time)

### install latest version of php

install these repositories to get the versions of php

`dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm` and 

`dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm`

After the successful installation of yum-utils and Remi-packages, search for the PHP modules which are available for download by running the command.
 `dnf module list php` to get the list of php versions`

The output indicates that the currently installed version of PHP is PHP 7.2. To install the newer release, PHP 7.4, reset the PHP modules.

`dnf module reset php`

Having reset the PHP modules, enable the PHP 7.4 module by running.

`dnf module enable php:remi-7.4`

Finally, install PHP, PHP-FPM (FastCGI Process Manager) and associated PHP modules using the command.

`dnf install php php-opcache php-gd php-curl php-mysqlnd`

We then check the php version to confirm `php -v` (we should get version ^7.4)

we need to start and enable PHP-FPM on boot-up.

`systemctl start php-fpm`

`systemctl enable php-fpm`

To check its status execute the command.

`systemctl status php-fpm`

To instruct SELinux to allow Apache to execute the PHP code via PHP-FPM run.

`setsebool -P httpd_execmem 1`

Finally, start Apache web server for PHP to work with Apache web server.

`systemctl start httpd`

![apache](https://github.com/femie15/darey/blob/main/project%201/project6/16-php-install.PNG)

![apache](https://github.com/femie15/darey/blob/main/project%201/project6/17-redhat.PNG)

### security group

![apache](https://github.com/femie15/darey/blob/main/project%201/project6/18-db-securityGroup.PNG)

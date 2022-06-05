# AWS CLOUD SOLUTION FOR 2 COMPANY WEBSITES USING A REVERSE PROXY TECHNOLOGY

## Architechture

![apache](https://github.com/femie15/darey/blob/main/project%201/project15/archi.png)

WARNING: delete all resources after practice to avoid cost.

Configuring AWS account and Organization Unit 

- Create an AWS Master account. (Also known as Root Account)
- Within the Root account, create a sub-account and name it femie15. (needing another email address to complete)
- Within the Root account, create an AWS Organization Unit (OU). Name it Dev. (We will launch Dev resources in there)
- Move the DevOps account into the Dev OU.
- Login to the newly created AWS account using the new email address.

![apache](https://github.com/femie15/darey/blob/main/project%201/project15/1-account.PNG)

- Create a free domain name for your fictitious company at Freenom domain registrar here.

![apache](https://github.com/femie15/darey/blob/main/project%201/project15/2-freenom.PNG)

- Create a hosted zone in AWS, and map it to your free domain from Freenom. Watch how to do that here

![apache](https://github.com/femie15/darey/blob/main/project%201/project15/3-nameserver.PNG)

### VPC

- Create a VPC with the name of the company (with IPv4 CIDR of 10.0.0.0/16), 

![apache](https://github.com/femie15/darey/blob/main/project%201/project15/4-vpc.PNG)

- Edit the DNS Hostname and enable it.

![apache](https://github.com/femie15/darey/blob/main/project%201/project15/5-dnsHN.PNG)

- We now create an Internet Gateway and attach it to a VPC

![apache](https://github.com/femie15/darey/blob/main/project%201/project15/6-internet_gateway.PNG)

![apache](https://github.com/femie15/darey/blob/main/project%201/project15/7-attach.PNG)

- now we create a subnets as shown in the architecture with different availability zones. (for the 2public subnet we used IPv4 CIDR 10.0.0.0/24 and 10.0.2.0/24 [even numbers] while for the 4 private subnets, we used 10.0.1.0/24, 10.0.3.0/24, 10.0.5.0/24, 10.0.7.0/24). Ensure a pair of Private subnets are in the same availability zones as each publlic subnet as specified in the architectural diagram.

![apache](https://github.com/femie15/darey/blob/main/project%201/project15/8-pub-subnet.PNG)

![apache](https://github.com/femie15/darey/blob/main/project%201/project15/9-private.PNG)

- Create a route (browt-pub-rt) table and associate it with public subnets (check the route table and click on "Subnet associations" then "Edit Subnet associations" to map it with pulic subnets)

- Create a route (browt-priv-rt) table and associate it with private subnets and do the same as above.

- Edit a route in public route table, and make it have access to the public internet (0.0.0.0/0)  make the target the Internet Gateway. (This is what allows a public subnet to be accessible from the Internet)

![apache](https://github.com/femie15/darey/blob/main/project%201/project15/11-editroute.PNG)

![apache](https://github.com/femie15/darey/blob/main/project%201/project15/12-route.PNG)

- Create 3 Elastic IPs and allocate it to a resource (Nat gateway)...

![apache](https://github.com/femie15/darey/blob/main/project%201/project15/13-elasticIP.PNG)

- Create a Nat Gateway and assign one of the Elastic IPs (*The other 2 will be used by Bastion hosts)

![apache](https://github.com/femie15/darey/blob/main/project%201/project15/14-Nat.PNG)

Now we can revisit our "route table" by Editing a route in private route table ("action" > "Edit route"), and associate it with the NAT-Gateway as Target and "0.0.0.0/0" as Destination.

![apache](https://github.com/femie15/darey/blob/main/project%201/project15/15-RT.PNG)

So, in summary, the public subnet route to the internet gateway (public /external internet access) while the private subnet routes to the NAT gateway (only allows outbound access)

-Create a Security Group for:

1. Nginx Server (Reverse proxy): Access to Nginx should only be allowed from a Application Load balancer (ALB) and SSH access to our bastion host as well. At this point, we have not created a load balancer, therefore we will update the rules later. For now, just create it and put some dummy records as a place holder.
2. Bastion Servers: We give SSH Access to the Bastion servers and should be allowed only from workstations that need to SSH into the bastion servers. Hence, you can use your workstation public IP address. To get this information, simply go to your terminal and type curl www.canhazip.com
3. Application Load Balancer (internal): Internal ALB will be available from the Nginx server only (HTTP / HTTPS)
4. Application Load Balancer (external): External ALB will be available from the Internet (HTTP / HTTPS)
5. Webservers: HTTPS/HTTP Access to Webservers should only be allowed from the internal ALB. and SSH access from the bastion host only
6. Data Layer: Access to the Data layer, which is comprised of Amazon Relational Database Service (RDS) and Amazon Elastic File System (EFS) must be carefully desinged. We give "MYSQL/Aurora" access to the bastion host and webservers, and NFS access to webserver as well.

### Certificate Manager

We need to create a certificate for our domain name (after registering and inputing nameservers).

- Navigate to the Amazon Certificate Manager service page
- Click "Request a certificate"
- check "Request a public certificate" and click next
- at the domain name field, use a wild card `*.ccchf.ml` to cater for subsequent sub-domains.
- use DNS validation 
- Give a "Name" tag, and click "Request".
- Then click the requested certificate and click on "Create Record in Route 53" and it will automatically write in the R53.

### EFS (Amazon- Elastic File System)

- Navigate to the EFS on console and click "create"
- give it a name and select our VPC and "Regional" availability
- Click on "customize"  to have more options
- Give a tag name and click next
- For network config; select our new VPC, for mount target, select private-subnet-1 and private-subnet-2 and the security group will be the "data-layer" (where the EFS is situated)
- Click "next" and "Create"

### Access Point

- Click on the newly created EFS and sselect "access point" then click "Create access point"
- Give the AP a name e.g. "wordpress"
- Root directory path "/wordpress"
- User ID (POSIX user ID), Group ID, Owner user ID, Owner group ID,  are all "0" (meaning root user)
- POSIX permissions to apply to the root directory path is "0755" (read and write permission)
- Give it a tagname "wordpress-AP"
- Do the same thing for "tooling" as well.

### Key Management Service

- On console goto KMS and click "create Key"
- create the key by giving it a name and selecting our account as administrator, then Allow administrator to delete and use the key

### RDS

- Navigate to RDS and click "Create subnet group"
- give it a name, select the VPC and the private subnet 3 & 4 (to host the RDS according to the diagram)
- then click create.

________________________________________________________________________

- Goto Database and create
- You should check for price difference to inform your decision on the type and tier category.(we will use the free tier for the testing purpose- the major disadvantage is that we wont be able to select our created key management service for dB encryption)
- select "standard", "Mysql", prefered version, "free tier", "DB instance identifier", username & password, "Virtual private cloud (VPC)", "Subnet group", set "Public access" to "NO", "Existing VPC security groups" to "data-layer", "Availability Zone", "password autherntication".
- Click "create dB"

### configure compute

Note: to create an auto-scaling group we need AMI, launch template (to create ec2 instances including the bastion host) & target group (Nginx (Reverse proxy)) - attached to a load balancer.

create 3 red hat instances for: bastion,nginx and webserver

### use these installation for bastion host ( <a href="https://github.com/femie15/ACS-project-config/blob/main/Installation.md"> reference </a>)

```
yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm

yum install wget vim python3 telnet htop git mysql net-tools chrony -y

systemctl start chronyd

systemctl enable chronyd
```

### Set Up Compute Resources for Nginx ( <a href="https://github.com/femie15/ACS-project-config/blob/main/Installation.md"> reference </a>)

```
yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm

yum install wget vim python3 telnet htop git mysql net-tools chrony -y

systemctl start chronyd

systemctl enable chronyd
```

to install the below

```
python
ntp
net-tools
vim
wget
telnet
epel-release
htop
```

configure selinux policies for the webservers and nginx servers

```
setsebool -P httpd_can_network_connect=1
setsebool -P httpd_can_network_connect_db=1
setsebool -P httpd_execmem=1
setsebool -P httpd_use_nfs 1
```

Install amazon efs utils for mounting the target on the Elastic file system

```
git clone https://github.com/aws/efs-utils

cd efs-utils

yum install -y make

yum install -y rpm-build

make rpm 

yum install -y  ./build/amazon-efs-utils*rpm
```

type `cd` (to come out of the entered "utils" directory)

seting up self-signed certificate for the nginx instance

```
sudo mkdir /etc/ssl/private

sudo chmod 700 /etc/ssl/private

openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/ACS.key -out /etc/ssl/certs/ACS.crt

sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048
```

Note the following ;

_______________________________________________

Country Name (2 letter code) [XX]:UK

State or Province Name (full name) []:London

Locality Name (eg, city) [Default City]:London

Organization Name (eg, company) [Default Company Ltd]:Browt

Organizational Unit Name (eg, section) []:DevOps

Common Name (eg, your name or your server's hostname) []:ip-172-31-26-210.ec2.internal

Email Address []:name@gmail.com
___________________________________________

### Webserver


```
yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm

yum install wget vim python3 telnet htop git mysql net-tools chrony -y

systemctl start chronyd

systemctl enable chronyd
```

```
setsebool -P httpd_can_network_connect=1
setsebool -P httpd_can_network_connect_db=1
setsebool -P httpd_execmem=1
setsebool -P httpd_use_nfs 1
```


```
git clone https://github.com/aws/efs-utils

cd efs-utils

yum install -y make

yum install -y rpm-build

make rpm 

yum install -y  ./build/amazon-efs-utils*rpm
```

seting up self-signed certificate for the apache webserver instance

```
yum install -y mod_ssl

openssl req -newkey rsa:2048 -nodes -keyout /etc/pki/tls/private/ACS.key -x509 -days 365 -out /etc/pki/tls/certs/ACS.crt

vi /etc/httpd/conf.d/ssl.conf
```

edit : `SSLCertificateFile /etc/pki/tls/certs/ACS.crt` and `SSLCertificateKeyFile /etc/pki/tls/private/ACS.key`

### Create an AMI out of the EC2 instance

- select on the webserver on console
- click "Actions > Image and Templates > Create Image"
- fill the form  and create image

Do the same for bastion and nginx

We can check the AMIs by going to Ami on the console...

### Target Groups

- Create Target groups
- Make Instance a target type, give it a name
- select HTTPS protocol
- select our created VPC
- Protocol version - HTTP1
- health check (HTTPS) - "/healthstatus"
- add a tag name
- Click "next" then "Create target group"

### TG for wordpress & Tooling

-repeat the steps above for wordpress & Tooling targets

### Load Balancers (Ext ad Int)

- Goto Load balancer and follow the configuration by selecting our VPCs, Public subnets, Nginx-target group (for External LB), Ext-ALB-SG, Listeners & routing : Nginx (HTTPS), 
- It automatically picks our Default SSL/TLS certificate (ACM).
- Create Load Balancer
- Repeat the process for the Internal Load balancer. (use wordpress as default route)

### configure tooling

- Click on the tooling LB
- Check the default listener and  under "RULES" Click "View/edit rules"
- click on the + sign, then "insert rules".
- Save

### lunch template

- Goto lunch template and create for bastion,nginx, webserver

bastion userdata:

```
#!/bin/bash 
yum install -y mysql 
yum install -y git tmux 
yum install -y ansible
```

for nginx userdata:

```
#!/bin/bash
yum install -y nginx
systemctl start nginx
systemctl enable nginx
git clone https://github.com/femie15/ACS-project-config.git
mv /ACS-project-config/reverse.conf /etc/nginx/
mv /etc/nginx/nginx.conf /etc/nginx/nginx.conf-distro
cd /etc/nginx/
touch nginx.conf
sed -n 'w nginx.conf' reverse.conf
systemctl restart nginx
rm -rf reverse.conf
rm -rf /ACS-project-config
```

Nginx Reverse.conf

Note: `server_name  *.ccchf.ml;` and ` proxy_pass https://internal-BR-Int-lb-1262810285.us-east-1.elb.amazonaws.com; ` (gotten from internal LB DNS name)

for wordpress userdata:

goto EFS console, click on access point, select the wordpress and click "attach" 

copy the test and paste below (3rd line)

```
#!/bin/bash
mkdir /var/www/
sudo mount -t efs -o tls,accesspoint=fsap-05faa6cbe4a70d3be fs-01be6aa8558198905:/ /var/www/
yum install -y httpd 
systemctl start httpd
systemctl enable httpd
yum module reset php -y
yum module enable php:remi-7.4 -y
yum install -y php php-common php-mbstring php-opcache php-intl php-xml php-gd php-curl php-mysqlnd php-fpm php-json
systemctl start php-fpm
systemctl enable php-fpm
wget http://wordpress.org/latest.tar.gz
tar xzvf latest.tar.gz
rm -rf latest.tar.gz
cp wordpress/wp-config-sample.php wordpress/wp-config.php
mkdir /var/www/html/
cp -R /wordpress/* /var/www/html/
cd /var/www/html/
touch healthstatus
sed -i "s/localhost/brdb.cizxwiigpcpq.us-east-1.rds.amazonaws.com/g" wp-config.php 
sed -i "s/username_here/ACSadmin/g" wp-config.php 
sed -i "s/password_here/admin12345/g" wp-config.php 
sed -i "s/database_name_here/wordpressdb/g" wp-config.php 
chcon -t httpd_sys_rw_content_t /var/www/html/ -R
systemctl restart httpd
```

for tooling userdata:

```
#!/bin/bash
mkdir /var/www/
sudo mount -t efs -o tls,accesspoint=fsap-0afc46fc243685fce fs-01be6aa8558198905:/ /var/www/
yum install -y httpd 
systemctl start httpd
systemctl enable httpd
yum module reset php -y
yum module enable php:remi-7.4 -y
yum install -y php php-common php-mbstring php-opcache php-intl php-xml php-gd php-curl php-mysqlnd php-fpm php-json
systemctl start php-fpm
systemctl enable php-fpm
git clone https://github.com/femie15/tooling.git
mkdir /var/www/html
cp -R /tooling-1/html/*  /var/www/html/
cd /tooling-1
mysql -h acs-database.cdqpbjkethv0.us-east-1.rds.amazonaws.com -u admin -p toolingdb < tooling-db.sql
cd /var/www/html/
touch healthstatus
sed -i "s/$db = mysqli_connect('mysql.tooling.svc.cluster.local', 'admin', 'admin12345', 'toolingdb');/$db = mysqli_connect('brdb.cizxwiigpcpq.us-east-1.rds.amazonaws.com', 'admin', 'admin12345', 'toolingdb');/g" functions.php
chcon -t httpd_sys_rw_content_t /var/www/html/ -R
systemctl restart httpd
```

### Auto scaling group

complete the auto scaling group, create database "wordpressdb" and "toolingdb" and test.



















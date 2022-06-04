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

- Create an AMI out of the EC2 instance


### Prepare Launch Template For Nginx (One Per Subnet)

Make use of the AMI to set up a launch template

Ensure the Instances are launched into a public subnet

Assign appropriate security group

Configure Userdata to update yum package repository and install nginx

Configure Target Groups

Select Instances as the target type

Ensure the protocol HTTPS on secure TLS port 443

Ensure that the health check path is /healthstatus

Register Nginx Instances as targets

Ensure that health check passes for the target group

Configure Autoscaling For Nginx

Select the right launch template

Select the VPC

Select both public subnets

Enable Application Load Balancer for the AutoScalingGroup (ASG)

Select the target group you created before

Ensure that you have health checks for both EC2 and ALB

The desired capacity is 2

Minimum capacity is 2

Maximum capacity is 4

Set scale out if CPU utilization reaches 90%

Ensure there is an SNS topic to send scaling notifications

Set Up Compute Resources for Bastion

Provision the EC2 Instances for Bastion

Create an EC2 Instance based on CentOS Amazon Machine Image (AMI) per each Availability Zone in the same Region and same AZ where you created Nginx server

Ensure that it has the following software installed

python
ntp
net-tools
vim
wget
telnet
epel-release
htop

### Associate an Elastic IP with each of the Bastion EC2 Instances

Create an AMI out of the EC2 instance

Prepare Launch Template For Bastion (One per subnet)

Make use of the AMI to set up a launch template

Ensure the Instances are launched into a public subnet

Assign appropriate security group

Configure Userdata to update yum package repository and install Ansible and git

Configure Target Groups

Select Instances as the target type

Ensure the protocol is TCP on port 22

Register Bastion Instances as targets

Ensure that health check passes for the target group

Configure Autoscaling For Bastion

Select the right launch template

Select the VPC

Select both public subnets

Enable Application Load Balancer for the AutoScalingGroup (ASG)

Select the target group you created before

Ensure that you have health checks for both EC2 and ALB

The desired capacity is 2

Minimum capacity is 2

Maximum capacity is 4

Set scale out if CPU utilization reaches 90%

Ensure there is an SNS topic to send scaling notifications

Set Up Compute Resources for Webservers

Provision the EC2 Instances for Webservers

Now, you will need to create 2 separate launch templates for both the WordPress and Tooling websites

Create an EC2 Instance (Centos) each for WordPress and Tooling websites per Availability Zone (in the same Region).

Ensure that it has the following software installed

python
ntp
net-tools
vim
wget
telnet
epel-release
htop
php
Create an AMI out of the EC2 instance

### Prepare Launch Template For Webservers (One per subnet)

Make use of the AMI to set up a launch template

Ensure the Instances are launched into a public subnet

Assign appropriate security group

Configure Userdata to update yum package repository and install wordpress (Only required on the WordPress launch template)

TLS Certificates From Amazon Certificate Manager (ACM)

You will need TLS certificates to handle secured connectivity to your Application Load Balancers (ALB).

Navigate to AWS ACM

Request a public wildcard certificate for the domain name you registered in Freenom

Use DNS to validate the domain name

Tag the resource

## Application Load Balancer To Route Traffic To NGINX

- Create a ALB
- Ensure that it listens on HTTPS protocol (TCP port 443)
- Ensure the ALB is created within the appropriate VPC | AZ | Subnets
- Choose the Certificate from ACM
- Select Security Group
- Select Nginx Instances as the target group

## Internal Load Balancer

- Create an Internal ALB
- Ensure that it listens on HTTPS protocol (TCP port 443)
- Ensure the ALB is created within the appropriate VPC | AZ | Subnets
- Choose the Certificate from ACM
- Select Security Group
- Select webserver Instances as the target group
- Ensure that health check passes for the target group

## Setup EFS

- Create an EFS filesystem
- Create an EFS mount target per AZ in the VPC, associate it with both subnets dedicated for data layer
- Associate the Security groups created earlier for data layer.
- Create an EFS access point. (Give it a name and leave all other settings as default)

## Setup RDS

Create a KMS key from Key Management Service (KMS) to be used to encrypt the database instance.

- Create a subnet group and add 2 private subnets (data Layer)
- Create an RDS Instance for mysql 8.*.*
- To satisfy our architectural diagram, you will need to select either Dev/Test or Production Sample Template. (But to minimize AWS cost, you can select the Do not create a standby instance option under Availability & durability sample template (The production template will enable Multi-AZ deployment)
Configure other settings accordingly (For test purposes, most of the default settings are good to go). In the real world, you will need to size the database appropriately. You will need to get some information about the usage. If it is a highly transactional database that grows at 10GB weekly, you must bear that in mind while configuring the initial storage allocation, storage autoscaling, and maximum storage threshold.
- Configure VPC and security (ensure the database is not available from the Internet)
- Configure backups and retention
- Encrypt the database using the KMS key created earlier
- Enable CloudWatch monitoring and export Error and Slow Query logs (for production, also include Audit)

Configuring DNS with Route53

Create other records such as CNAME, alias and A records.

Create an alias record for the root domain and direct its traffic to the ALB DNS name.

Create an alias record for tooling.<yourdomain>.com and direct its traffic to the ALB DNS name.

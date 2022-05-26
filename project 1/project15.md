# AWS CLOUD SOLUTION FOR 2 COMPANY WEBSITES USING A REVERSE PROXY TECHNOLOGY

## Architechture

![apache](https://github.com/femie15/darey/blob/main/project%201/project15/archi.png)

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

- now we create a subnets as shown in the architecture with different availability zones. (for the 2public subnet we used IPv4 CIDR 10.0.2.0/24 and 10.0.4.0/24 [even numbers] while for the 4 private subnets, we used 10.0.1.0/24, 10.0.4.0/24, 10.0.6.0/24, 10.0.8.0/24)

![apache](https://github.com/femie15/darey/blob/main/project%201/project15/8-pub-subnet.PNG)

![apache](https://github.com/femie15/darey/blob/main/project%201/project15/9-private.PNG)

- Create a route (browt-pub-rt) table and associate it with public subnets (check the route table and click on "Subnet associations" then "Edit Subnet associations" to map it with pulic subnets)

- Create a route (browt-priv-rt) table and associate it with private subnets and do the same as above.

- Edit a route in public route table, and associate it with the Internet Gateway. (This is what allows a public subnet to be accisble from the Internet)

![apache](https://github.com/femie15/darey/blob/main/project%201/project15/11-editroute.PNG)

- Create 3 Elastic IPs and allocate it to a resource (Nat gateway)...

![apache](https://github.com/femie15/darey/blob/main/project%201/project15/13-elasticIP.PNG)

- Create a Nat Gateway and assign one of the Elastic IPs (*The other 2 will be used by Bastion hosts)

![apache](https://github.com/femie15/darey/blob/main/project%201/project15/14-Nat.PNG)

Now we can revisit our "route table" by Editing a route in private route table ("action" > "Edit route"), and associate it with the NAT-Gateway as Target and "0.0.0.0/0" as Destination.

![apache](https://github.com/femie15/darey/blob/main/project%201/project15/15-RT.PNG)

-Create a Security Group for:

1. Nginx Servers: Access to Nginx should only be allowed from a Application Load balancer (ALB). At this point, we have not created a load balancer, therefore we will update the rules later. For now, just create it and put some dummy records as a place holder.
2. Bastion Servers: Access to the Bastion servers should be allowed only from workstations that need to SSH into the bastion servers. Hence, you can use your 
3. workstation public IP address. To get this information, simply go to your terminal and type curl www.canhazip.com
4. Application Load Balancer: ALB will be available from the Internet
5. Webservers: Access to Webservers should only be allowed from the Nginx servers. Since we do not have the servers created yet, just put some dummy records as a place holder, we will update it later.
6. Data Layer: Access to the Data layer, which is comprised of Amazon Relational Database Service (RDS) and Amazon Elastic File System (EFS) must be carefully desinged – only webservers should be able to connect to RDS, while Nginx and Webservers will have access to EFS Mountpoint.

### Set Up Compute Resources for Nginx

- Provision EC2 Instances for Nginx
- Create an EC2 Instance based on CentOS Amazon Machine Image (AMI) in any 2 Availability Zones (AZ) in any AWS Region (Use EC2 instance of T2 family)
- Ensure that it has the following software installed:
- 
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

# Apache load balancer

### Continuation from project 7

![apache](https://github.com/femie15/darey/blob/main/project%201/project8/archi.PNG)

### Configure Apache As A Load Balancer

Create an Ubuntu Server 20.04 EC2 instance and name it Project-8-apache-lb

![apache](https://github.com/femie15/darey/blob/main/project%201/project8/1-servers.PNG)

Open TCP port 80 on Project-8-apache-lb by creating an Inbound Rule in Security Group.

Install Apache Load Balancer on Project-8-apache-lb server and configure it to point traffic coming to LB to both Web Servers

### Install apache2
` apt update`
` apt install apache2 -y `
` apt-get install libxml2-dev`

### Enable following modules:
` a2enmod rewrite `
` a2enmod proxy `
` a2enmod proxy_balancer `
` a2enmod proxy_http `
` a2enmod headers` 
` a2enmod lbmethod_bytraffic `

### Restart apache2 service
` systemctl restart apache2` 

Make sure apache2 is up and running

`sudo systemctl status apache2`

## Configure load balancing

` vi /etc/apache2/sites-available/000-default.conf`

### Add this configuration into this section <VirtualHost *:80> .... </VirtualHost>

`<Proxy "balancer://mycluster">
               BalancerMember http://<WebServer1-Private-IP-Address>:80 loadfactor=5 timeout=1
               BalancerMember http://<WebServer2-Private-IP-Address>:80 loadfactor=5 timeout=1
               ProxySet lbmethod=bytraffic
               # ProxySet lbmethod=byrequests
        </Proxy>

        ProxyPreserveHost On
        ProxyPass / balancer://mycluster/
        ProxyPassReverse / balancer://mycluster/`

![apache](https://github.com/femie15/darey/blob/main/project%201/project8/2-LB.PNG)

![apache](https://github.com/femie15/darey/blob/main/project%201/project8/3-LB%20web.PNG)

### Restart apache server

` systemctl restart apache2`

If in the Project-7 you mounted "/var/log/httpd/" from your Web Servers to the NFS server "/mnt/logs" ,
– unmount them and make sure that each Web Server has its own log directory.

`umount -l /var/log/httpd` (-l lazy unmount)

create error_log and access_log file in all web servers "/var/log/httpd" directory

`touch /var/log/httpd/error_log` and `touch /var/log/httpd/access_log`

then restart the web servers 

`systemctl restart httpd`

Open two ssh/Putty/terminal consoles for both Web Servers and run following command:

`tail -f /var/log/httpd/access_log`

![apache](https://github.com/femie15/darey/blob/main/project%201/project8/4-log1.PNG)

![apache](https://github.com/femie15/darey/blob/main/project%201/project8/5-log2.PNG)

### Configure Local DNS Names Resolution

Sometimes it is tedious to remember and switch between IP addresses, especially if you have a lot of servers under your management.
What we can do, is to configure local domain name resolution. 

The easiest way is to use /etc/hosts file, although this approach is not very scalable, but it is very easy to configure and shows the concept well. So let us configure IP address to domain name mapping for our LB.

Open this file on your LB server

`vi /etc/hosts`

Add 2 records into this file with Local IP address and arbitrary name for both of your Web Servers

<WebServer1-Private-IP-Address> Web1

<WebServer2-Private-IP-Address> Web2

Now you can update your LB config file with those names instead of IP addresses.

BalancerMember http://Web1:80 loadfactor=5 timeout=1

BalancerMember http://Web2:80 loadfactor=5 timeout=1

You can try to curl your Web Servers from LB locally curl http://Web1 or curl http://Web2


# Congratulations

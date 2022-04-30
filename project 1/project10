# NGINX load balancer

![apache](https://github.com/femie15/darey/blob/main/project10/archi.PNG)

### Free URL

Goto https://my.freenom.com/domains.php -> Register new domain

![apache](https://github.com/femie15/darey/blob/main/project10/1-domainName.PNG)

the type a domain name with a free extention e.g. ccchf.tk in the space and select checkout.

then follow the instruction to finish the registration.

### Configure NGINX As A Load Balancer

Spin up an Ubuntu server and ssh into it.then perform these.

`apt update && apt install nginx -y` then

![apache](https://github.com/femie15/darey/blob/main/project10/2-install.PNG)

`systemctl enable nginx && systemctl start nginx` (this is to make the nginx server to auto start with the server instance.

We need to create a configuration file for our load balancer.

`vi /etc/nginx/sites-available/load_balancer.conf`

and paste the config below...

`#insert following configuration into http section
 upstream myproject {
    server 172.31.13.12 weight=5;
    server 172.31.15.162 weight=5;
  }
server {
    listen 80;
    server_name ccchf.tk www.ccchf.tk;
    location / {
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_pass http://web;
    }
  }
#comment out this line
#       include /etc/nginx/sites-enabled/*;`

![apache](https://github.com/femie15/darey/blob/main/project10/3-config.PNG)

we can then remove the default nginx configuration 

`rm -f /etc/nginx/sites-enabled/default`

We now need to confirm if the nginx is successfully configured by 

`nginx -t`

now we currently dont have any file in the site-enabled directory. go into the directory `cd /etc/nginx/sites-enabled`

we can now link our  "/etc/nginx/sites-available/load_balancer.conf" to "/etc/nginx/sites-enabled" so that nginx can access the configuration.

`ln -s ../sites-available/load_balancer.conf .` (The last "." means "in this directory")

![apache](https://github.com/femie15/darey/blob/main/project10/4-link.PNG)

now when we do `ls` we can now view the load_balancer.conf file. then reload Nginx.

`systemctl reload nginx`

We can then visit our domain name from our browser and it should load our website.

we now need to make it secure.

Install certbot and request for an SSL/TLS certificate. Make sure snapd service is active and running

`systemctl status snapd`

### Install certbot

`apt install certbot -y` or `snap install --classic certbot -y`

`apt install python3-certbot-nginx -y`

(Request your certificate (just follow the certbot instructions – you will need to choose which domain you want your certificate to be issued for, domain name will be looked up from nginx.conf file so make sure you have updated it earlier).

`ln -s /snap/bin/certbot /usr/bin/certbot` [this is neccessary if we are using snap to install certbot].
`certbot --nginx` )

we can check our config and reload nginx now.

`nginx -t && nginx -s reload`

Once everything is okay, we need to create a certificate for out domain. on the terminal,

`certbot --nginx -d ccchf.tk -d www.ccchf.tk`

a prompt requesting for email address will be up, we ca then enter an email address. we then press enter on keyboard and we are prompted to read terms and conditions.
we can pree "A" to agree.

another prompt to share email which we can choose "yes" or "no", afterwards it takes sometime to create the certificate.

we now get a prompt to either redirect to secure https access, we select "2" to redirect all incoming request from port 80 to 443.

we then see a success message, we then go to the website to view if the configuration of certificate works well.

![apache](https://github.com/femie15/darey/blob/main/project10/5-web.PNG)

### Cron Job setting for renewal 

You can test renewal command in dry-run mode

`certbot renew --dry-run`

Best pracice is to have a scheduled job that to run renew command periodically. Let us configure a cronjob to run the command twice a day.

To do so, lets edit the crontab file with the following command:

`crontab -e`

the terminal then brings a prompt to choose the editor, we pick "2" for vim

Add following line:

`* */12 * * *   root /usr/bin/certbot renew > /dev/null 2>&1`

the format of this line is "m h dom mon dow  command", "/dev/null" means it is not creating a log file

You can always change the interval of this cronjob if twice a day is too often by adjusting schedule expression.


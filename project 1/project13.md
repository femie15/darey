# ANSIBLE DYNAMIC ASSIGNMENTS (INCLUDE) AND COMMUNITY ROLES

Static assignments use "import" Ansible module. The module that enables dynamic assignments is "include".

In the https://github.com/<your-name>/ansible-config-mgt GitHub repository start a new branch and call it dynamic-assignments.

Create a new folder, name it "dynamic-assignments". Then inside this folder, create a new file and name it "env-vars.yml".
  
Since we will be using the same Ansible to configure multiple environments, and each of these environments will have certain unique attributes, such as servername, ip-address etc., 
we will need a way to set values to variables per specific environment.
  
To keep each environment’s variables file. Create a new folder "env-vars", then for each environment, create new YAML files which we will use to set variables.
  
The layout should now look like this.
  

  
Now enter these into the "env-vars.yml" file
  
```
  ---
- name: collate variables from env specific file, if it exists
  hosts: all
  tasks:
    - name: looping through list of available files
      include_vars: "{{ item }}"
      with_first_found:
        - files:
            - dev.yml
            - stage.yml
            - prod.yml
            - uat.yml
          paths:
            - "{{ playbook_dir }}/../env-vars"
      tags:
        - always 
```
  
  
### 3 things worth noting
  
1. From Ansible version 2.8, the include module is deprecated and variants of include_* must be used. These are:
  
```
include_role
include_tasks
include_vars 
```
  
In the same version, variants of import were also introduces, such as:

import_role
import_tasks
  
2. We made use of a special variables { playbook_dir } and { inventory_file }. { playbook_dir } will help Ansible to determine the location of the running playbook, and from there navigate to other path on the filesystem. { inventory_file } on the other hand will dynamically resolve to the name of the inventory file being used, then append .yml so that it picks up the required file within the env-vars folder.

3. We are including the variables using a loop. with_first_found implies that, looping through the list of files, the first one found is used. This is good so that we can always set default values in case an environment specific env file does not exist.

### Update site.yml with dynamic assignments
 
Update "site.yml" file to make use of the dynamic assignment.
  
```
  ---
- hosts: all
- name: Include dynamic variables 
  tasks:
  import_playbook: ../static-assignments/common.yml 
  include: ../dynamic-assignments/env-vars.yml
  tags:
    - always

-  hosts: webservers
- name: Webserver assignment
  import_playbook: ../static-assignments/webservers.yml
```

On Jenkins-Ansible server make sure that git is installed with "git --version", then go to ‘ansible-config-artifact’ directory and run

```
git init
git branch -m master main (if master was the default branch created on the local server) 
git pull https://github.com/femie15/ansible-config-mgt.git main
git remote add origin https://github.com/femie15/ansible-config-mgt.git
git branch roles-feature
git checkout roles-feature
```
  
(if the git is not working from the root directory goto `vi .ssh/authorized_keys` copy the ssh key and add the key to github ssh keys. then redo the process.)
  
Inside roles `cd roles` directory create your new MySQL role with `ansible-galaxy install geerlingguy.mysql` and rename the folder to mysql

`mv geerlingguy.mysql/ mysql` then edit `vi mysql/defaults/main.yml` and edit the following
  
```
 # Databases.
 mysql_databases:
    - name: tooling
      collation: utf8_general_ci
      encoding: utf8
      replicate: 1

 # Users.
 mysql_users:
    - name: webaccess
      host: 0.0.0.0.
      password: secret
      priv: '*.*:ALL,GRANT'
```

Now it is time to upload the changes into your GitHub:

```
git add .
git commit -m "Commit new role files into GitHub"
git checkout main
git merge roles-feature
git remote add origin https://github.com/femie15/ansible-config-mgt.git
git push origin
```

### LOAD BALANCER ROLES
  
We want to be able to choose which Load Balancer to use, Nginx or Apache, so we need to have two roles respectively:

1. Nginx - `ansible-galaxy install geerlingguy.nginx`
  
2. Apache - `ansible-galaxy install geerlingguy.apache`
 
then we can rename the directory `mv geerlingguy.nginx/ nginx` and `mv geerlingguy.apache/ apache`
  
then edit `vi nginx/defaults/main.yml` and note the following and change
  
```
nginx_upstreams:[]
# - name: myapp1
#   strategy: "ip_hash" # "least_conn", etc.
#   keepalive: 16 # optional
#   servers:
#     - "srv1.example.com"
#     - "srv2.example.com weight=3"
#     - "srv3.example.com"
```
  
  to
  
```
nginx_upstreams:
 - name: myapp1
   strategy: "ip_hash" # "least_conn", etc.
   keepalive: 16 # optional
   servers:
     - "172.31.85.40 weight=3"
     - "172.31.90.83 weight=3"
```
  
  and
  
```
nginx_extra_http_options: ""
# Example extra http options, printed inside the main server http config:
#    nginx_extra_http_options: |
#      proxy_buffering    off;
#      proxy_set_header   X-Real-IP $remote_addr;
#      proxy_set_header   X-Scheme $scheme;
#      proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
#      proxy_set_header   Host $http_host;
```
  
  to
  
```
nginx_extra_http_options:
 Example extra http options, printed inside the main server http config:
    nginx_extra_http_options: |
      proxy_buffering    off;
      proxy_set_header   X-Real-IP $remote_addr;
      proxy_set_header   X-Scheme $scheme;
      proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header   Host $http_host;
```

goto `vi nginx/tasks/main.yml` 
  
and insert `become: true` to the nginx setup.
  
  
and also remove the "Rocky server option"
  


  
  
  
Since we cannot use both Nginx and Apache load balancer, we need to add a condition to enable either one (this is where you can make use of variables).

Declare a variable in defaults/main.yml file inside the Nginx and Apache roles. Name each variables enable_nginx_lb and enable_apache_lb respectively.

Set both values to false like this enable_nginx_lb: false and enable_apache_lb: false.

Declare another variable in both roles load_balancer_is_required and set its value to false as well

Update both assignment and site.yml files respectively

loadbalancers.yml file

- hosts: lb
  roles:
    - { role: nginx, when: enable_nginx_lb and load_balancer_is_required }
    - { role: apache, when: enable_apache_lb and load_balancer_is_required }
site.yml file

     - name: Loadbalancers assignment
       hosts: lb
         - import_playbook: ../static-assignments/loadbalancers.yml
        when: load_balancer_is_required 
  
  
  
  
  
  
  
  
  
  
  
  
  

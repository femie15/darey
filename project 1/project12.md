# ANSIBLE REFACTORING AND STATIC ASSIGNMENTS (IMPORTS AND ROLES)

### Purpose: enhance Project 11 by introducing a new Jenkins project/job (we will require "Copy Artifact" plugin)

Go to your Jenkins-Ansible server and create a new directory called ansible-config-artifact – we will store there all artifacts after each build.

`mkdir /home/ubuntu/ansible-config-artifact` (note the present working directory with `pwd` to know how to structure the directory path)

Change permissions to this directory, so Jenkins could (access and) save files there  

`chmod -R 0777 /home/ubuntu/ansible-config-artifact`

Go to Jenkins "web console -> Manage Jenkins -> Manage Plugins" -> on "Available tab" search for "Copy Artifact" and install this plugin without restarting Jenkins

Create a "new item" (Freestyle project) and name it "save_artifacts".

This project will be triggered by completion of your existing ansible project. Configure it accordingly:

You can configure number of builds to keep in order to save space on the server, for example, you might want to keep only last 2 or 5 build results. You can also make this change to your ansible job.

The main idea of "save_artifacts" (item name) project is to save artifacts into "/home/ubuntu/ansible-config-artifact" directory. 
To achieve this, create a Build step and choose "Copy artifacts" from other project, specify ansible as a source project and "/home/ubuntu/ansible-config-artifact" as a target directory.

 You can configure number of builds to keep in order to save space on the server, for example, you might want to keep only last 2 or 5 build results. You can also make this change to your ansible job.

Test your set up by making some change in README.MD file inside your ansible-config-mgt repository (right inside master branch).
If both Jenkins jobs have completed one after another – you shall see your files inside "/home/ubuntu/ansible-config-artifact" directory and it will be updated with every commit to your master branch.

## REFACTOR ANSIBLE CODE BY IMPORTING OTHER PLAYBOOKS INTO SITE.YML

Ensure that you have pulled down the latest code from master (main) branch, and created a new branch, name it refactor.
In Project 11 you wrote all tasks in a single playbook common.yml, now it is pretty simple set of instructions for only 2 types of OS, but imagine you have many more tasks and you need to apply this playbook to other servers with different requirements. Breaking tasks up into different files is an excellent way to organize complex sets of tasks and reuse them.

Within playbooks folder, create a new file and name it "site.yml" – This file will now be considered as an entry point into the entire infrastructure configuration. Other playbooks will be included here as a reference. In other words, "site.yml" will become a parent to all other playbooks that will be developed. Including "common.yml" that you created previously.

Create a new folder in root of the repository and name it "static-assignments". The static-assignments folder is where all other children playbooks will be stored.

Move "common.yml" file into the newly created "static-assignments" folder.

Inside "site.yml" file, import "common.yml" playbook.

`---
- hosts: all
- import_playbook: ../static-assignments/common.yml`

The code above uses built in import_playbook Ansible module.

### Run ansible-playbook command against the dev environment

Since you need to apply some tasks to your dev servers and wireshark is already installed
 you can go ahead and create another playbook under static-assignments and name it "common-del.yml". In this playbook, configure deletion of wireshark utility.

```
---
- name: update web, nfs and db servers
  hosts: webservers, nfs
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
  - name: delete wireshark
    yum:
      name: wireshark
      state: removed

- name: update LB server
  hosts: lb, db
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
  - name: delete wireshark
    apt:
      name: wireshark-qt
      state: absent
      autoremove: yes
      purge: yes
      autoclean: yes
      ```

update "site.yml" with "- import_playbook: ../static-assignments/common-del.yml" instead of "- import_playbook: ../static-assignments/common.yml" and run it against dev servers:

`cd /home/ubuntu/ansible-config-mgt/` (refactor branch)

do `git add.` to add all changes, `git commit -m "delete wireshark"` to commit the changes, `git push --set-upstream origin refactor` to set the new upstream "refactor", now the changes is available and we can do a merge to the main branch.

now we have our "ansible-config-artifact" directory filled with the neccessary files and playbook.

### we need to point our ansible config file to the inventory directory

so we go to the ansible config file `vi /etc/ansible/ansible.cfg` and edit (point) the inventory directory to "/home/ubuntu/ansible-config-artifact/inventory"

now run `ansible all -m ping` to ping all the listed hosts if they are reachable.

after successfully pinging the hosts, 

ensure you are in the directory "/home/ubuntu/ansible-config-artifact" and run `ansible-playbook -i inventory/dev.yml playbooks/site.yml`

Make sure that wireshark is deleted on all the servers by ssh-ing into the servers and running `wireshark --version` or `which wireshark` on the servers.

## CONFIGURE UAT WEBSERVERS WITH A ROLE ‘WEBSERVER’

We have our nice and clean dev environment, so let us put it aside and configure 2 new Web Servers as uat. We could write tasks to configure Web Servers in the same playbook, but it would be too messy, instead, we will use a dedicated role to make our configuration reusable.

Launch 2 fresh EC2 instances using RHEL 8 image, we will use them as our uat servers, so give them names accordingly – Web1-UAT and Web2-UAT.



to create a role, you must create a directory called "roles/", relative to the playbook file or in "/etc/ansible/" directory.
There are two ways how you can create this folder structure:

1. Use an Ansible utility called "ansible-galaxy" inside "ansible-config-artifact/roles" directory (you need to create roles directory upfront)
`mkdir roles`
`cd roles` run `ansible-galaxy init webserver`

2. Create the directory/files structure manually

Note: You can choose either way, but since you store all your codes in GitHub, it is recommended to create folders and files there rather than locally on Jenkins-Ansible server.

The entire folder structure should look like below, but if you create it manually – you can skip creating tests, files, and vars or remove them if you used ansible-galaxy (in ansible server)

Update your inventory "ansible-config-artifact/inventory/uat.yml" file with IP addresses of your 2 UAT Web servers

`[uat-webservers]
<Web1-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user' 
<Web2-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user' `
 
### Note: Ensure the ansible server can ssh into the uat web servers.

### The webserver directory must be created in the "roles" directory.

In "/etc/ansible/ansible.cfg" file uncomment "roles_path" string and provide a full path to your roles directory "roles_path = /home/ubuntu/ansible-config-artifact/roles", so Ansible could know where to find configured roles.

It is time to start adding some logic to the webserver role. Go into "tasks" directory, and within the "main.yml" file, start writing configuration tasks to do the following:

1. Install and configure Apache (httpd service)
2. Clone Tooling website from GitHub https://github.com/femie15/tooling.git
3. Ensure the tooling website code is deployed to /var/www/html on each of 2 UAT Web servers.
4. Make sure httpd service is started

```
---
- name: install apache
  become: true
  ansible.builtin.yum:
    name: "httpd"
    state: present

- name: install git
  become: true
  ansible.builtin.yum:
    name: "git"
    state: present

- name: clone a repo
  become: true
  ansible.builtin.git:
    repo: https://github.com/<your-name>/tooling.git
    dest: /var/www/html
    force: yes

- name: copy html content to one level up
  become: true
  command: cp -r /var/www/html/html/ /var/www/

- name: Start service httpd, if not started
  become: true
  ansible.builtin.service:
    name: httpd
    state: started

- name: recursively remove /var/www/html/html/ directory
  become: true
  ansible.builtin.file:
    path: /var/www/html/html
    state: absent
```

###Reference ‘Webserver’ role
 
Within the "static-assignments" folder, create a new assignment(file) for uat-webservers "uat-webservers.yml". This is where you will reference the role. paste the code below.
 
 ```
 ---
- hosts: uat-webservers
  roles:
     - webserver
 ```
 
The entry point to our ansible configuration is the "site.yml" file. Therefore, you need to refer your "uat-webservers.yml" role inside "site.yml".
So, we should have this in "site.yml"
 
```
 ---
- hosts: all
- import_playbook: ../static-assignments/common.yml

- hosts: uat-webservers
- import_playbook: ../static-assignments/uat-webservers.yml
 ```
 
Now commit your changes, create a Pull Request and merge them to master branch, make sure webhook triggered two consequent Jenkins jobs, they ran successfully and copied all the files to your Jenkins-Ansible server into "/home/ubuntu/ansible-config-artifact/" directory.

now run `ansible all -m ping` to ping all the listed hosts if they are reachable. (the UAT servers should be up now).
 
Now run the playbook against your uat inventory 
 
`ansible-playbook -i /home/ubuntu/ansible-config-artifact/inventory/uat.yml /home/ubuntu/ansible-config-artifact/playbooks/site.yml`
 
and see what happens:
 
You should be able to see both of your UAT Web servers configured and you can try to reach them from your browser:

"http://<Web1-UAT-Server-Public-IP-or-Public-DNS-Name>/index.php" or "http://<Web2-UAT-Server-Public-IP-or-Public-DNS-Name>/index.php"
 
If the redhat default page is showing, then we can configure it to automatically route to index.php page.
 
Your Ansible architecture now looks like this:


 
 
 
 







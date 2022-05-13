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

`---
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
      autoclean: yes`

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













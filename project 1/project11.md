# ANSIBLE

![apache](https://github.com/femie15/darey/blob/main/project11/archi.PNG)

### installation

From the continuation of project 10 (Multiple web servers, NGINX load balancer, Jenkins, NFS server and a database server)

We can also install our ansible on the same jenkins server (Rename the server to jenkins-ansible) and SSH into it. We then install ansible.

`sudo apt update`

`sudo apt install ansible`

After the installation is done, check your Ansible version by running `ansible --version`

then we go to our jenkins GUI and create a "new item" give it a name of "ansible" and select "free-style project" then click "ok"

in the next view, under the Source Code Management tab, click "git" and enter github https url for the ansible repository, then change the branch to "*/main".

under Build Triggers check the box "GitHub hook trigger for GITScm polling", then click "Post-build Actions" and select "Archive the artifacts" and in the  "Files to archive"
field enter "**" (save all files). Now click save to save the configuration then on the side navigation bar, click "Build now", The first build should be successful now,
You can go ahead to change the Readme.txt file on github and it should automatically build on jenkins.

make sure that builds starts automatically and Jenkins saves the files (build artifacts) in following folder
`ls /var/lib/jenkins/jobs/ansible/builds/<build_number>/archive/` (eg. `ls /var/lib/jenkins/jobs/ansible/builds/2/archive`)

Now we have the initial architechture (In the first image) setup.

### Note:

Every time you stop/start your Jenkins-Ansible server – you have to reconfigure GitHub webhook to a new IP address, in order to avoid it, it makes sense to allocate an Elastic IP to your Jenkins-Ansible server (you have done it before to your LB server in Project 10). Note that Elastic IP is free only when it is being allocated to an EC2 Instance, so do not forget to release Elastic IP once you terminate your EC2 Instance.

## Dev environment using VS Code

on your local machine, Clone down your ansible-config-mgt repo to your Jenkins-Ansible instance

`git clone <ansible-config-mgt repo link>` (advisable to use the ssh link)

confirm your branch with `git main` then run `git status` to view the status

Note: Give your branches descriptive and comprehensive names, for example, if you use Jira as a project management tool 
– include ticket number (e.g. proj-11) in the name of your branch and add a topic and a brief description what this branch is about – a bugfix, hotfix, feature, release (e.g. feature/proj-11-ansible)

`git checkout -b proj-11`

Create a directory and name it "playbooks" – it will be used to store all your playbook files.
Create a directory and name it "inventory" – it will be used to keep your hosts organised.
Within the playbooks folder, create your first playbook, and name it "common.yml"
Within the inventory folder, create an inventory file (.yml) for each environment (Development, Staging Testing and Production) dev, staging, uat, and prod respectively.
To create folders use 
`mkdir playbooks` 

to create files use 
`touch dev.yml staging.yml uat.yml prod.yml` (for linux / mac)
`type nul > dev.yml` (for windows)

### Set up an Ansible Inventory

An Ansible inventory file defines the hosts and groups of hosts upon which commands, modules, and tasks in a playbook operate. 
Since our intention is to execute Linux commands on remote hosts, and ensure that it is the intended configuration on a particular server that occurs. 
It is important to have a way to organize our hosts in such an Inventory.

Exit the remote server from the terminal, and run these commands.

Note: Ansible uses TCP port 22 by default, which means it needs to ssh into target servers from Jenkins-Ansible host – for this you can implement the concept of ssh-agent. Now you need to import your key into ssh-agent:

(To learn how to setup SSH agent and connect VS Code to your Jenkins-Ansible instance, please see this video:
For Windows users – <a href="https://youtu.be/OplGrY74qog">ssh-agent on windows</a>
For Linux users – <a href="https://youtu.be/OplGrY74qog">ssh-agent on linux</a>)

from your local machine move the ssh key to the ansible server

`scp -i femie.pem femie.pem ubuntu@54.226.126.86:~/shell`

Now, ssh into your Jenkins-Ansible server using ssh-agent
the file was uploaded with name "shell", now rename to "femie.pem" `mv shell femie.pem`
and run these

" eval `ssh-agent -s` "
`ssh-add femie.pem`

Confirm the key has been added with the command below, you should see the name of your key

`ssh-add -l` 

`ssh -A ubuntu@public-ip` from the local into the jenkins server, 

`ssh -A ubuntu@private-ip` from the jenkins server into other servers (the pem file made this possible.

The Load Balancer user is ubuntu and user for RHEL-based servers is ec2-user.

Update your inventory/dev.yml file with this snippet of code:

`[nfs]
<NFS-Server-Private-IP-Address> ansible_ssh_user='ec2-user'

[webservers]
<Web-Server1-Private-IP-Address> ansible_ssh_user='ec2-user'
<Web-Server2-Private-IP-Address> ansible_ssh_user='ec2-user'

[db]
<Database-Private-IP-Address> ansible_ssh_user='ec2-user' 

[lb]
<Load-Balancer-Private-IP-Address> ansible_ssh_user='ubuntu'`

### Ansible instructions

It is time to start giving Ansible the instructions on what you needs to be performed on all servers listed in inventory/dev.

In common.yml playbook you will write configuration for repeatable, re-usable, and multi-machine tasks that is common to systems within the infrastructure.

Update your playbooks/common.yml file with following code:

`---
- name: update web, nfs and db servers
  hosts: webservers, nfs, db
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
    - name: ensure wireshark is at the latest version
      yum:
        name: wireshark
        state: latest

- name: update LB server
  hosts: lb
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
    - name: Update apt repo
      apt: 
        update_cache: yes

    - name: ensure wireshark is at the latest version
      apt:
        name: wireshark
        state: latest`

### Update GIT with the latest code

use git commands to add, commit and push your branch to GitHub.

`git status`

`git add <selected files>` e.g. `git add .` Note: "." means all

`git commit -m "commit message" `
  
`git push origin proj-11`
  
Create a Pull request (PR) on github

Wear a hat of another developer for a second, and act as a reviewer.

If the reviewer is happy with your new feature development, merge the code to the master branch.
  
Once the merge is completed on github, jenkins will start building, we can now see the new builds both on jenkins and from our terminal.
  
### terminal
  
Head back on your terminal, checkout from the feature branch into the master, and pull down the latest changes.
  
`git checkout main`
  
and run `git pull` to maake our main branch up to date with the origin.
  
 We should have a remote explorer extention on our VS Code (search microsofts' "Remote Development pack")
  
 ### Run ansible 
  
 confirm if you can connect to the servers from the jenkins server 
  
`ssh ubuntu@<private IP>` for Ubuntu servers and `ssh ec2-user@<private IP>` RHEL servers
  
ansible-playbook -i /var/lib/jenkins/jobs/ansible/builds/4/archive/inventory/dev.yml /var/lib/jenkins/jobs/ansible/builds/4/archive/playbooks/common.yml

afterwards you can go to each of the servers and check if wireshark has been installed by running `which wireshark` or `wireshark --version`

the updated with Ansible architecture now looks like this:

 ## Congratulations !!!

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
















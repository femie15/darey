# AUTOMATE INFRASTRUCTURE WITH IAC USING TERRAFORM. PART 4 – TERRAFORM CLOUD


### Terraform cloud

Create a Terraform Cloud account

At this stage you need to <a href="https://app.terraform.io/public/signup/account">create a terraform account</a> , or simply <a href="https://app.terraform.io/session">login</a> if you do have one.

![apache](https://github.com/femie15/darey/blob/main/project%201/project19/1-login.PNG)

Create an organization

Select "Start from scratch", choose a name for your organization and create it.

![apache](https://github.com/femie15/darey/blob/main/project%201/project19/2-orgWorkspace.PNG)

Configure a workspace

Push your Terraform codes developed in the previous projects to a new repository called "terraform-cloud". in the workspace configuration, Choose version control workflow and you will be promped to connect your GitHub account to your workspace follow the prompt and add your newly created repository to the workspace.

![apache](https://github.com/femie15/darey/blob/main/project%201/project19/3-github.PNG)

![apache](https://github.com/femie15/darey/blob/main/project%201/project19/4-githubAccess.PNG)

Move on to "Configure settings", provide a description for your workspace and in the advance settings check "Automatic speculative plans" to enable terraform to run a plan before merging to the master branch, click "Create workspace".

![apache](https://github.com/femie15/darey/blob/main/project%201/project19/5-wksps.PNG)

Configure variables

Terraform Cloud supports two types of variables: 
1. environment variables 
2. Terraform variables. (they are both sensitive)

Set two environment variables: AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY, set the values that you used in Project 16. These credentials will be used to privision your AWS infrastructure by Terraform Cloud.

![apache](https://github.com/femie15/darey/blob/main/project%201/project19/6-variables.PNG)

we dont need to specify the terraform variables so far we have the "terraform.auto.tfvars" file in place.

### Packer & Ansible

Ensure you have packer and Ansible installed (via chocolatey for windows or homebrew for mac) on your local machine then add the following two files in the existing reprository from projest 18.

- AMI: for building packer images
- Ansible: for Ansible scripts to configure the infrastucture

add the neccesary packer files (each packer file for each sh file), run `packer build .\bastion.pkr.hcl`, this will generate a new AMI for the bastion and create a keypair, ec2 instace etc.

![apache](https://github.com/femie15/darey/blob/main/project%201/project19/7-packer.PNG)

![apache](https://github.com/femie15/darey/blob/main/project%201/project19/8-pkey.PNG)

Do the same for "nginx.pkr.hcl", "ubuntu.pkr.hcl" and "web.pkr.hcl".

![apache](https://github.com/femie15/darey/blob/main/project%201/project19/9-ami.PNG)

Update the "terraform.auto.tfvars" file with the new "AMIs" created with packer and then push the code to github.

from web console, run terraform plan and terraform apply

Switch to "Runs" tab and click on "start a new plan",  "Queue plan manualy" button. If planning has been successfull, you can proceed and confirm Apply – press "Confirm and apply", provide a comment and "Confirm plan"

Check the logs and verify that everything has run correctly. Note that Terraform Cloud has generated a unique state version that you can open and see the codes applied and the changes made since the last run.

Test automated terraform plan and see if it works fine.

configure ansible manually by connecting to the instance and ensure you allow ansible to bet the updated ip address irrespective of when the auto-scaling group take an instance out of service. ensure you use teraform output (eg ip address) to update ansible

Update the "nginx.conf.j2" file in template inside ansible-nginx, and set the proxy-pass to the latest one from aws.

also update "setup-db.yaml" file in tooling-task to the terraform endpoint, dbname, user and password.

copy a mount access point into the source field in task -> "main.yaml" file. we then go ahead to run ansible config file then ansible playbook.

now we can restart our nginx and check the website which should be up. and we are able to login.

![apache](https://github.com/femie15/darey/blob/main/project%201/project19/11-wordpress.PNG)

# Done

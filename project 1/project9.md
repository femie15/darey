# CONTINOUS INTEGRATION PIPELINE FOR TOOLING WEBSITE

### Continuation from project 8

![apache](https://github.com/femie15/darey/blob/main/project%201/project9/archi.PNG)

### Install Jenkins server

Create an AWS EC2 server based on Ubuntu Server 20.04 LTS and name it "Jenkins"

Install JDK (since Jenkins is a Java-based application)

` apt update`

` apt install default-jdk-headless`

Install Jenkins

`wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -`

` sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
    /etc/apt/sources.list.d/jenkins.list'`
` apt update`

` apt-get install jenkins`

Make sure Jenkins is up and running

` systemctl status jenkins`

By default Jenkins server uses TCP port 8080 – open it by creating a new Inbound Rule in your EC2 Security Group

Perform initial Jenkins setup.
From your browser access http://<Jenkins-Server-Public-IP-Address-or-Public-DNS-Name>:8080

You will be prompted to provide a default admin password, do `cat /var/lib/jenkins/secrets/initialAdminPassword` to retrive the password and then enter it in the web browser
and follow the promts by filling the form and then reaching the dashboard.

### configure a simple Jenkins job/project

We need to goto github repo and select "settings" then "webhooks", Click "New webhook" and enter Payload-URL "http://54.226.126.86:8080/github-webhook/"

change content-type to "application/json" and save.

Go to Jenkins web console, click "New Item" and create a "Freestyle project" by clicking "ok"
To connect your GitHub repository, you will need to provide its URL, you can copy from the repository (https url) itself

Save the configuration and let us try to run the build. For now we can only do it manually.
Click "Build Now" button, if you have configured everything correctly, the build will be successfull and you will see it under #1

### Configure triggering the job from GitHub webhook:

go back to the project, Click "Configure" and click "Build Triggers"
and check check "GitHub hook trigger for GITScm polling"

Configure "Post-build Actions" to archive all the files – files resulted from a build are called "artifacts".

Now, go ahead and make some change in any file in your GitHub repository (e.g. README.MD file) and push the changes to the master branch.
You will see that a new build has been launched automatically (by webhook) and you can see its results – artifacts, saved on Jenkins server.

You have now configured an automated Jenkins job that receives files from GitHub by webhook trigger (this method is considered as ‘push’ because the changes are being ‘pushed’ and files transfer is initiated by GitHub). There are also other methods: trigger one job (downstreadm) from another (upstream), poll GitHub periodically and others.
By default, the artifacts are stored on Jenkins server locally

`ls /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/` i.e. "ls /var/lib/jenkins/jobs/project9/builds/2/archive/"

## CONFIGURE JENKINS TO COPY FILES TO NFS SERVER VIA SSH
  
Now we have our artifacts saved locally on Jenkins server, the next step is to copy them to our NFS server to /mnt/apps directory.

Jenkins is a highly extendable application and there are 1400+ plugins available. We will need a plugin that is called "Publish Over SSH".

Install "Publish Over SSH" plugin.
On main dashboard select "Manage Jenkins" and choose "Manage Plugins" menu item.

On "Available" tab search for "Publish Over SSH" plugin and install it by checking the box before it and clicking "install without refresh"
  
### Configure the job/project to copy artifacts over to NFS server.
  
go back to the dashboard and click "manage jenkins" then "configuration system" and look for the new "Publish Over SSH" plugin
  
  
copy the content of the SSH key we use to login to our instance into the key field, then type "NFS" in the Name field, Hostname has "<Private-IP-address-of-the-NFS-server>"

  Username field "ec2-user", and remote directory field "/mnt/apps".
  
  Click "test configuration" to give a "Success" test response, then click "Save".
  
  Save the configuration, open your Jenkins job/project configuration page and add another one "Post-build Action"
  
  Save this configuration and go ahead, change something in README.MD file in your GitHub Tooling repository.

Webhook will trigger a new job and in the "Console Output" of the job you will find something like this:

"SSH: Transferred 25 file(s)
Finished: SUCCESS
To make sure that the files in /mnt/apps have been udated – connect via SSH/Putty to your NFS server and check README.MD file"

`cat /mnt/apps/README.md`
  
If you see the changes you had previously made in your GitHub – the job works as expected.
  
  ## Congratulations
  

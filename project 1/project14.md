# ANSIBLE ROLES FOR CI ENVIRONMENT

Spin up a new RedHat server (t2medium preferably for more capacity). Then install git and jenkins by following these steps

Git:

`yum install git -y` we can then clone-down the ansible-config repo.

Jenkins: visit jenkins installation page

Install wget; `yum install wget -y`

download jenkins repo: `wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo`

download jenkins key: `rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key`

Now, goto the "Readme.md" file of the ansible-config repo and install epel and remi realeases

`yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm`

`yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm`

Now, install Java

`yum install java-11-openjdk-devel -y`

open the bash profile

`vi .bash_profile`

paste the below in the bash profile (to export the stated path for java at system start-up so we dont have to do that again)

```
PATH=$PATH:$HOME/bin
export PATH

export JAVA_HOME=$(dirname $(dirname $(readlink $(readlink $(which javac)))))
export PATH=$PATH:$JAVA_HOME/bin
export CLASSPATH=.:$JAVA_HOME/jre/lib:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar
```

reload the bash profile

`source ~/.bash_profile`

install jenkins;

`yum install jenkins` and run `systemctl daemon-reload`

we can then start jenkins, enable it and check the status with these commands `systemctl start jenkins`, `systemctl enable jenkins`, `systemctl status jenkins`

we can then proceed to our jenkins server url "<publicIpAddress>:8080" (ensure access to port 8080 is allowed in the security group)

once we see the jenkins page, we can follow the instructions (install suggested plugins e.t.c.), and then arrive at the dashboard.

We can now search for "blue ocean" plugin from the available plugins, and create a pipeline from our "ansible-config" github repo. through our GUI (this may require us to also create a github token)

At this point you may not have a Jenkinsfile in the Ansible repository, so Blue Ocean will attempt to give you some guidance to create one. But we do not need that. We will rather create one ourselves. We now need to click on administration on the jenkins interface to exit blue ocean console.
  
Click dashboard to view our newly created pipeline. It takes the name of the GitHub repository.

We now need to create a jenkins file so as to contain all codes for our deployment.
  
Inside the Ansible project, create a new directory, name it "deploy" and start a new file "Jenkinsfile" inside the directory, then copy the snippet.
  
```
pipeline {
    agent any

  stages {
    stage('Build') {
      steps {
        script {
          sh 'echo "Building Stage"'
        }
      }
    }
    }
}
```
  
the pipeline currently has just one stage called Build and the only thing we are doing is using the shell script module to echo Building Stage.
 
we need to push the change to github. `git add .` , `git commit -m "add Jenkinsfile` and `git push`. (If you have issues with authenticating from github, you can input your username in the respective field and input the personall access token in the password field).

Now go back into the Ansible pipeline in Jenkins web interface, goto the "ansible-config" project and select "configure" 

Scroll down to "Build Configuration" section and specify the location of the Jenkinsfile as "deploy/Jenkinsfile" and click "save"
  
Goback to the pipeline again "ansible-config", this time click "Build now". we should see the build initiating, and we can view more details in the blue ocean dashboard.
  
We can also try triggering the build again from Blue Ocean interface.
  
on dashboard, click on the project name, than Click on Blue Ocean and Click on the play button against the branch. If there were more than one branch in GitHub, Jenkins would have scanned the repository to discover them all and we would have been able to trigger a build for each branch.
  
### multi-branch
  
1. Create a new git branch and name it "feature/jenkinspipeline-stages" `git checkout -b feature/jenkinspipeline-stages`
  
2. Currently we only have the "Build" stage. Let us add another stage called "Test". Paste the code snippet below and push the new changes to GitHub.
  
```
   pipeline {
    agent any

  stages {
    stage('Build') {
      steps {
        script {
          sh 'echo "Building Stage"'
        }
      }
    }

    stage('Test') {
      steps {
        script {
          sh 'echo "Testing Stage"'
        }
      }
    }
    }
}
```

To make your new branch show up in Jenkins, we need to tell Jenkins to scan the repository. Click on the blue ocean "Administration" button 
  
  
Navigate to the Ansible project and click on "Scan repository now"
  
  
Refresh the page and both branches will start building automatically. You can go into Blue Ocean and see both branches there too.

  
In Blue Ocean, you can now see how the Jenkinsfile has caused a new step in the pipeline launch build for the new branch

1. Create a pull request to merge the latest code into the main branch
2. After merging the PR, go back into your terminal and switch into the main branch.
3. Pull the latest change.
4. Create a new branch, add more stages into the Jenkins file to simulate below phases. (Just add an echo command like we have in build and test stages)
   1. Package 
   2. Deploy 
   3. Clean up (we add this stage to delete the created directory in "/var/lib/jenkins/workspace"

5. Verify in Blue Ocean that all the stages are working, then merge your feature branch to the main branch
  
 ________________________________________________________________________________________________________________________________ 
  
 ### RUNNING ANSIBLE PLAYBOOK FROM JENKINS
  

  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  






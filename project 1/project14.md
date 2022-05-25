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
  
1. Installing Ansible on Jenkins server 
  
`yum install ansible -y` and confirm by `ansible --version`
  
Now we need to install some ansible dependencies.
  
```
python3 -m pip install --upgrade setuptools
python3 -m pip install --upgrade pip
python3 -m pip install PyMySQL
python3 -m pip install mysql-connector-python
python3 -m pip install psycopg2==2.7.5 --ignore-installed
```
Community installation
  
For Mysql Database `ansible-galaxy collection install community.mysql`
  
For Postgresql Database `ansible-galaxy collection install community.postgresql`
  
2. Installing Ansible plugin in Jenkins UI

Goto Dashboard > manage jenkins > manage plugins > click "available" and search for "ansible" then install without reloading.
   
____________________________________________________________________________________________________________________________________________
### Possible errors to watch out for:

1. Ensure that the git module in Jenkinsfile is checking out SCM to main branch instead of master (GitHub has discontinued the use of Master due to Black Lives Matter. You can read more here)

2. Jenkins needs to export the ANSIBLE_CONFIG environment variable. You can put the ".ansible.cfg" `touch .ansible.cfg` file alongside Jenkinsfile in the deploy directory (though we have a default configuration file in "/etc/ansible/ansible.cfg" but we need to define our own). we can then paste the snippet below.
  
```
[defaults]
timeout = 160
callback_whitelist = profile_tasks
log_path=~/ansible.log
host_key_checking = False
gathering = smart
ansible_python_interpreter=/usr/bin/python3
allow_world_readable_tmpfiles=true

[ssh_connection]
ssh_args = -o ControlMaster=auto -o ControlPersist=30m -o ControlPath=/tmp/ansible-ssh-%h-%p-%r -o ServerAliveInterval=60 -o ServerAliveCountMax=60 -o ForwardAgent=yes
```

This way, anyone can easily identify that everything in there relates to deployment. Then, using the Pipeline Syntax tool in Ansible, generate the syntax to create environment variables to set.

and then set the global workspace in our "Jenkinsfile"  (remember that ${WORKSPACE} represents the directory "/var/lib/jenkins/workspace/<branchName>")

```
environment {
      ANSIBLE_CONFIG="${WORKSPACE}/deploy/ansible.cfg"
    }
```
 
### Possible issues to watch out for when you implement this

1. Remember that ansible.cfg must be exported to environment variable so that Ansible knows where to find Roles. But because you will possibly run Jenkins from different git branches, the location of Ansible roles will change. Therefore, you must handle this dynamically. You can use Linux Stream Editor sed to update the section roles_path each time there is an execution. You may not have this issue if you run only from the main branch.

2. If you push new changes to Git so that Jenkins failure can be fixed. You might observe that your change may sometimes have no effect. Even though your change is the actual fix required. This can be because Jenkins did not download the latest code from GitHub. Ensure that you start the Jenkinsfile with a clean up step to always delete the previous workspace before running a new one. Sometimes you might need to login to the Jenkins Linux server to verify the files in the workspace to confirm that what you are actually expecting is there. Otherwise, you can spend hours trying to figure out why Jenkins is still failing, when you have pushed up possible changes to fix the error.

3. Another possible reason for Jenkins failure sometimes, is because you have indicated in the Jenkinsfile to check out the main git branch, and you are running a pipeline from another branch. So, always verify by logging onto the Jenkins box to check the workspace, and run git branch command to confirm that the branch you are expecting is there.
  
___________________________________________________________________________________________________________________________________________
  

3. Creating "Jenkinsfile" from scratch. (We will delete all the current configuration and start all over to get Ansible to run successfully)
  
Here we will create 2 new instances (for Nginx and Db)

Now we can finish up our "Jenkinsfile" with the stages.

```
parameters {
      string(name: 'inventory', defaultValue: 'dev',  description: 'This is the inventory file for the environment to deploy configuration')
    }

  stages{
      stage("Initial cleanup") {
          steps {
            dir("${WORKSPACE}") {
              deleteDir()
            }
          }
        }

      stage('Checkout SCM') {
         steps{
            git branch: 'feature/jenkinspipeline-stages', url: 'https://github.com/femie15/ansible-config.git'
         }
       }

      stage('Prepare Ansible For Execution') {
        steps {
          sh 'echo ${WORKSPACE}' 
          sh 'sed -i "3 a roles_path=${WORKSPACE}/roles" ${WORKSPACE}/deploy/ansible.cfg'  
        }
     }

      stage('Run Ansible playbook') {
        steps {
           ansiblePlaybook become: true, colorized: true, credentialsId: 'private-key', disableHostKeyChecking: true, installation: 'ansible', inventory: 'inventory/${inventory}', playbook: 'playbooks/site.yml'
         }
      }

      stage('Clean Workspace after build'){
        steps{
          cleanWs(cleanWhenAborted: true, cleanWhenFailure: true, cleanWhenNotBuilt: true, cleanWhenUnstable: true, deleteDirs: true)
        }
      }
   }
```

### Note: ansible need to ssh into our new instances

We can generate our "ansiblePlaybook" from jenkins GUI. Goto. Dashboard > manage jenkins > manage credentials > global > add credentials.
  
select "ssh Username with private key" , call ID anyname like "pk or private-key", username will be "ec2-user" (because we are using red hat), in the private ket section, click "Enter directly" and open up the .pem file for the instance, then paste the content in the space and leave passphrase as empty and click "OK".

We now go to dashboard > manage jenkins > Global Tool Configuration > Add Ansible
  
Name is  "ansible" and for the path (we can go to the terminal and run `which ansible` to get it) "/usr/bin/" then we save.
    
Go back to the project page and select "Pipeline Syntax" on the side navigation bar. we will then see the "Ansible tool" and the path to the playbook (for the project) is "playbooks/site.yml" , inventory filr path workspace is "inventory/dev". We can then select SSH connection credentials "ec2-user..", Check "use become", check "Disable the host SSH key check" and "colourized output" then click "Generate Pipeline Script" and copy the generated script and paste it in the "run ansible playbook" step in our "Jenkinsfile".
  
Add the appropriate IP address in the dev inventory. and run our ansible playbook via jenkins.
  
Now we need to introduce parameters in our "Jenkinsfile"
  
### Parameterizing Jenkinsfile For Ansible Deployment
  
Update Jenkinsfile to introduce parameterization
  
```
parameters {
      string(name: 'inventory', defaultValue: 'dev',  description: 'This is the inventory file for the environment to deploy configuration')
    }
```
  
we now replace "inventory/dev" which was hardcoded with `${inventory}, and it then requests for an input when triggered.
We can now push and merge to main.
  
## CI/CD PIPELINE FOR TODO APPLICATION
  
### Prepare Jenkins
Fork the repository below into your GitHub account and clone it `git clone https://github.com/darey-devops/php-todo.git`
  
On you Jenkins server, install PHP, its dependencies and Composer tool 
  
`sudo apt install -y zip libapache2-mod-php phploc php-{xml,bcmath,bz2,intl,gd,mbstring,mysql,zip}`
  
```
yum module reset php -y
yum module enable php:remi-7.4 -y
yum install -y php php-common php-mbstring php-opcache php-intl php-xml php-gd php-curl php-mysqlnd php-fpm php-json
systemctl start php-fpm
systemctl enable php-fpm
```

Then composer
  
`curl -sS https://getcomposer.org/installer | php`
  
`sudo mv composer.phar /usr/bin/composer`
  
to verify  `composer --version`
  
We now need to install "plot", "Artifactory" plugins in jenkins.

We will use plot plugin to display tests reports, and code coverage information.
The Artifactory plugin will be used to easily upload code artifacts into an Artifactory server.

We need to create an instance for the Artifactory server and use the ip address in the "ci" inventory file. and also reflect the change in the playbook. and push the update to github, then create PR and merge, and pull it from the main branch in jenkins.
  
we then goto our Jfrog web "<public IP>:8081" Username: admin and Password: password
  
set up new password and create local repo.
  
In Jenkins UI configure Artifactory, Configure the server ID, URL and Credentials, run Test Connection.

Integrate Artifactory repository with Jenkins
Create a dummy Jenkinsfile in the repository
Using Blue Ocean, create a multibranch Jenkins pipeline
On the database server, create database and user
  
``` 
Create database homestead;
CREATE USER 'homestead'@'%' IDENTIFIED BY 'sePret^i';
GRANT ALL PRIVILEGES ON * . * TO 'homestead'@'%';
```

Update the database connectivity requirements in the file ".env.sample"
Update "Jenkinsfile" with proper pipeline configuration
  
```
pipeline {
    agent any

  stages {

     stage("Initial cleanup") {
          steps {
            dir("${WORKSPACE}") {
              deleteDir()
            }
          }
        }

    stage('Checkout SCM') {
      steps {
            git branch: 'main', url: 'https://github.com/darey-devops/php-todo.git'
      }
    }

    stage('Prepare Dependencies') {
      steps {
             sh 'mv .env.sample .env'
             sh 'composer install'
             sh 'php artisan migrate'
             sh 'php artisan db:seed'
             sh 'php artisan key:generate'
      }
    }
  }
}
``` 
  
rename ".env.sample" to ".env"

Composer is used by PHP to install all the dependent libraries used by the application
php artisan uses the ".env" file to setup the required database objects – (After successful run of this step, login to the database, run show tables and you will see the tables being created for you)
  
### Update the Jenkinsfile to include Unit tests step

```
    stage('Execute Unit Tests') {
      steps {
             sh './vendor/bin/phpunit'
      } 
```

### Code Quality Analysis
  
we will use phploc, the data produced by phploc can be ploted onto graphs in Jenkins. Add the code analysis step in "Jenkinsfile". The output of the data will be saved in "build/logs/phploc.csv" file.  

Plot the data using plot Jenkins plugin.
  
Bundle the application code for into an artifact (archived package) upload to Artifactory
  
```
stage ('Package Artifact') {
    steps {
            sh 'zip -qr php-todo.zip ${WORKSPACE}/*'
     }
    }
``` 
  
Publish the resulted artifact into Artifactory
  
```
stage ('Upload Artifact to Artifactory') {
          steps {
            script { 
                 def server = Artifactory.server 'artifactory-server'                 
                 def uploadSpec = """{
                    "files": [
                      {
                       "pattern": "php-todo.zip",
                       "target": "<name-of-artifact-repository>/php-todo",
                       "props": "type=zip;status=ready"

                       }
                    ]
                 }""" 

                 server.upload spec: uploadSpec
               }
            }

        }
```
  
Deploy the application to the dev environment by launching Ansible pipeline
  
```  
stage ('Deploy to Dev Environment') {
    steps {
    build job: 'ansible-project/main', parameters: [[$class: 'StringParameterValue', name: 'env', value: 'dev']], propagate: false, wait: true
    }
  }  
```
  
The build job used in this step tells Jenkins to start another job. In this case it is the ansible-project job, and we are targeting the main branch.
  

  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  






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









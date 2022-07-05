# MIGRATION TO THE СLOUD WITH CONTAINERIZATION. PART 1 – DOCKER & DOCKER COMPOSE

1. Create a new Ubuntu EC2 instance to host our docker engine.
2. Update apt and Download the script to install the docker engine and run it with this command (Reference https://docs.docker.com/engine/install/ubuntu)

`sudo apt-get update`

Install packages to allow apt to use a repository over HTTPS:

```
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
```

Add Docker’s official GPG key:

`curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -` you will get an "ok" response

Verify that you now have the key with the fingerprint by searching for the last 8 characters of the fingerprint.

`sudo apt-key fingerprint 0EBFCD88`

`curl -fsSL https://get.docker.com -o get-docker.sh`

`sh ./get-docker.sh`

To verify if docker is installed run `sudo docker version` and a response on details of the docker will be displayed

### MYSQL

3. Pull Mysql image from docker hub

`docker pull mysql/mysql-server:latest`

you may be required to login, then use the command `docker login` and enter both username and password from dockerhub account

4. check to see the pulled image  `docker image ls`

5. Deploy the MySQL Container to Docker Engine

`docker run --name <container_name> -e MYSQL_ROOT_PASSWORD=<my-secret-pw> -d mysql/mysql-server:latest`

check to see if the MySQL container is running

`docker ps -a`

### Connecting to the MySQL Docker Container

Approach 1

Connecting directly to the container running the MySQL server:

`docker exec -it <container_name> bash`

or

`docker exec -it mysql mysql -uroot -p`

Provide the root password when prompted. With that, you’ve connected the MySQL client to the server.

______________________________________________________________________________________________________________________

Note:

exec used to execute a command from bash itself

-it makes the execution interactive and allocate a pseudo-TTY

bash this is a unix shell and its used as an entry-point to interact with our container

mysql The second mysql in the command "docker exec -it mysql mysql -uroot -p" serves as the entry point to interact with mysql container just like bash or sh

-u mysql username

-p mysql password

______________________________________________________________________________________________________________________


Approach 2

Stop and remove the previous mysql docker container. then check if it's still there

```
docker stop <container_name> 
docker rm <container_name>
or 
docker rm <container ID> 04a34f46fb98
docker ps -a
```

Now we carry out the following

1. Create a network:

`docker network create --subnet=172.18.0.0/24 tooling_app_network` and verify it with `docker network ls`

Docker will use the default network for all the containers run, we have created a "DRIVER Bridge" network which alsos serve as the default.
This will be neccessary if there is a requirement to control the cidr range of the containers running the entire application stack. This will be an ideal situation to create a network and specify the --subnet

2. Create an environment variable to store the root password we created earlier:

 `export MYSQL_PW=<MyPassword>`  and verify with `echo $MYSQL_PW`

3. Then, pull the image and run the container, all in one command :

`docker run --network tooling_app_network -h mysqlserverhost --name=mysql-server -e MYSQL_ROOT_PASSWORD=$MYSQL_PW  -d mysql/mysql-server:latest `

Flags used

-d runs the container in detached mode
--network connects a container to a network
-h specifies a hostname

Verify if the container is running `docker ps -a`

4. Create a file and name it "create_user.sql" and add the below code in the file:

`CREATE USER ''@'%' IDENTIFIED BY ''; GRANT ALL PRIVILEGES ON * . * TO ''@'%'; `

5. Run the script:

`docker exec -i mysql-server mysql -uroot -p$MYSQL_PW < create_user.sql `

If you see a warning like below, it is acceptable to ignore:

"mysql: [Warning] Using a password on the command line interface can be insecure."

6. Connecting to the MySQL server from a second container running the MySQL client utility

`docker run --network tooling_app_network --name mysql-client -it --rm mysql mysql -h mysqlserverhost -u  -p `

_______________________________________________________________________________

Flags used:

--name gives the container a name
-it runs in interactive mode and Allocate a pseudo-TTY
--rm automatically removes the container when it exits
--network connects a container to a network
-h a MySQL flag specifying the MySQL server Container hostname
-u user created from the SQL script
admin : username for user created from the SQL script "create_user.sql"
-p password specified for the user created from the SQL script

_______________________________________________________________________________

7. Clone the tooling repo 

`git clone https://github.com/darey-devops/tooling.git `

8. Export the location of the SQL file

`export tooling_db_schema=./tooling/html/tooling_db_schema.sql`

Verify that the path is exported `echo $tooling_db_schema`

9. Use the SQL script to create the database and prepare the schema (With the docker exec command, you can execute a command in a running container).

`docker exec -i mysql-server mysql -uroot -p$MYSQL_PW < $tooling_db_schema `

10. Update the ".env" file with connection details to the database

`sudo vi tooling/html/.env`

```
MYSQL_IP=mysqlserverhost
MYSQL_USER=username
MYSQL_PASS=client-secrete-password
MYSQL_DBNAME=toolingdb
```

_________________________________________________________________________________

Flags used:

MYSQL_IP mysql ip address "leave as mysqlserverhost"
MYSQL_USER mysql username for user export as environment variable
MYSQL_PASS mysql password for the user exported as environment varaible
MYSQL_DBNAME mysql databse name "toolingdb"

_________________________________________________________________________________

11. Run the Tooling App

Ensure you are inside the directory "tooling" that has the file Dockerfile `cd tooling` and build your container :

 `docker build -t tooling:0.0.1 .`
 
_________________________________________________________________________________
 
In the above command, we specify a parameter 

-t, so that the image can be tagged tooling"0.0.1 
Also, you have to notice the . at the end. This is important as that tells Docker to locate the Dockerfile in the current directory you are running the command. Otherwise, you would need to specify the absolute path to the Dockerfile.

_________________________________________________________________________________

Run the container:

`docker run --network tooling_app_network -p 8085:80 -it tooling:0.0.1 `

_________________________________________________________________________________

--network flag so that both the Tooling app and the database can easily connect on the same virtual network we created earlier.
The -p flag is used to map the container port with the host port. Within the container, apache is the webserver running and, by default, it listens on port 80
But we cannot directly use port 80 on our host machine because it is already in use. The workaround is to use another port that is not used by the host machine. In our case, port 8085 is free, so we can map that to port 80 running in the container.

_________________________________________________________________________________















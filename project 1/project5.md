# MYSQL Client-Server Setup

check into root of the mysql-server server and do `sudo -s`

update apt
`apt update -y`

install mysql server
`apt install mysql-server -y`

to confirm the installation `systemctl enable mysql` afterwards we then adjust the security group to allow access for the cliient server by getting the client server ip address `ip addr show` and pasting it in the mysql server SG.
![Security group](https://github.com/femie15/darey/blob/main/project%201/project%205/1-security%20group.PNG)



then check into root of the mysql-client server and do `sudo -s`

update apt
`apt update -y`

install mysql client
`apt install mysql-client -y`

we can the do the `mysql_secure_installation` action and may not neccesarily accept the first security prompt (due to test case)
then we set password etc.

then we login to mysql `mysql` (remember to add `sudo` before all command if not a root user)

we can the perform this

mysql> `CREATE USER 'remote_user'@'%' IDENTIFIED WITH mysql_native_password BY 'password';`

then

`CREATE DATABASE test_db;`

then we grant priviledge to the user.

`GRANT ALL ON test_db .* TO 'remote_user'@'%' WITH GRANT OPTION;`

Now we flush privileges
`FLUSH PRIVILEGES;`

we can exit now `exit;`

### we need to configure MySQL server to allow connections from remote hosts.
`vi /etc/mysql/mysql.conf.d/mysqld.cnf` in the file we replace the bind-address ‘127.0.0.1’ with ‘0.0.0.0’

and then restart the server

`systemctl restart mysql`

We can now go to the mysql-client server and connect to the mysql-server

`mysql -u remote_user -h 172.31.15.49 -p`

we are therefore prompted to enter password for the user `password` after then we can test some basic mysql queries.

`Show databases;`

We then get the list of databases.

## mysql-Client

![Client](https://github.com/femie15/darey/blob/main/project%201/project%205/3-client%20server.PNG)

## mysql-Server

![Security group](https://github.com/femie15/darey/blob/main/project%201/project%205/4-server.PNG)

# Hurray !!!

# WEB STACK IMPLEMENTATION (LEMP STACK)

## STEP 1 – INSTALLING THE NGINX WEB SERVER

`sudo apt update`

`sudo apt install nginx`

To verify that nginx was successfully installed and is running as a service in Ubuntu, run: 


`sudo systemctl status nginx`
![install-nginx](https://user-images.githubusercontent.com/33351638/146085018-875a50c1-473e-4067-84e0-91a975f73b14.JPG)

## STEP 2 — INSTALLING MYSQL
`sudo apt install mysql-server`

interactive script by running:

`sudo mysql_secure_installation`

test if you’re able to log in to the MySQL console by typing:

`sudo mysql`
![mysql](https://user-images.githubusercontent.com/33351638/146085036-db515207-a638-45f3-8962-5235f96ce498.JPG)

## Step 3 – Installing PHP

You’ll need to install php-fpm, which stands for “PHP fastCGI process manager”, and tell Nginx to pass PHP requests to this software for processing. Additionally, you’ll need php-mysql, a PHP module that allows PHP to communicate with MySQL-based databases. Core PHP packages will automatically be installed as dependencies.

To install these 2 packages at once, run:

`sudo apt install php-fpm php-mysql`
![php_install](https://user-images.githubusercontent.com/33351638/146084847-163d77fb-9016-4b13-8266-d6bd6c074dc2.JPG)
![php](https://user-images.githubusercontent.com/33351638/146085081-768e2d84-957b-48cd-b240-47b7b6ed68a7.JPG)


## STEP 4 — CONFIGURING NGINX TO USE PHP PROCESSOR

Create the root web directory for your_domain as follows:

`sudo mkdir /var/www/edumixlemp`

`sudo chown -R $USER:$USER /var/www/edumixlemp`
Then, open a new configuration file in Nginx’s sites-available directory using your preferred command-line editor. Here, we’ll use nano:

`sudo nano /etc/nginx/sites-available/edumixlemp`
This will create a new blank file. Paste in the following bare-bones configuration:

`#/etc/nginx/sites-available/edumixlemp

server {
    listen 80;
    server_name edumixlemp www.edumixlemp;
    root /var/www/edumixlemp;

    index index.html index.htm index.php;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
     }

    location ~ /\.ht {
        deny all;
    }

}`

Here’s what each of these directives and location blocks do:

### listen 
Defines what port Nginx will listen on. In this case, it will listen on port 80, the default port for HTTP.
### root
Defines the document root where the files served by this website are stored.
### index
Defines in which order Nginx will prioritize index files for this website. It is a common practice to list index.html files with a higher precedence than index.php files to allow for quickly setting up a maintenance landing page in PHP applications. You can adjust these settings to better suit your application needs.
### server_name 
Defines which domain names and/or IP addresses this server block should respond for. Point this directive to your server’s domain name or public IP address.
### location / 
The first location block includes a try_files directive, which checks for the existence of files or directories matching a URI request. If Nginx cannot find the appropriate resource, it will return a 404 error.
### location ~ \.php$ 
This location block handles the actual PHP processing by pointing Nginx to the fastcgi-php.conf configuration file and the php7.4-fpm.sock file, which declares what socket is associated with php-fpm.
### location ~ /\.ht 
The last location block deals with .htaccess files, which Nginx does not process. By adding the deny all directive, if any .htaccess files happen to find their way into the document root ,they will not be served to visitors.


Activate your configuration by linking to the config file from Nginx’s sites-enabled directory:

`sudo ln -s /etc/nginx/sites-available/edumixlemp /etc/nginx/sites-enabled/`
This will tell Nginx to use the configuration next time it is reloaded. You can test your configuration for syntax errors by typing:

`sudo nginx -t`

disable default Nginx host that is currently configured to listen on port 80, for this run:

`sudo unlink /etc/nginx/sites-enabled/default`

reload Nginx to apply the changes:

`sudo systemctl reload nginx`
![reload](https://user-images.githubusercontent.com/33351638/146084931-b3bb04ce-d16c-4bc8-bc2e-4008fcb1d678.JPG)

then write inside the index.html file
`sudo echo '<!DOCTYPE html>
<html>
<head>
<title>Welcome to EDUmix on Nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to EDUmix!</h1>
<p>best Education.</p>

<p><em>Thank you for using our service.</em></p>
</body>
</html>' > /var/www/edumixlemp/index.html`
![web](https://user-images.githubusercontent.com/33351638/146084968-0fea7d50-e05b-41cd-94af-ad4094e32c92.JPG)

## STEP 5 – TESTING PHP WITH NGINX

 test it to validate that Nginx can correctly hand .php files off to your PHP processor.
 Open a new file called info.php within your document root in your text editor:

`sudo nano /var/www/edumixlemp/info.php`

This is valid PHP code that will return information about your server:

`<?php
phpinfo();`

![phpinfo](https://user-images.githubusercontent.com/33351638/146084864-ddc05fd8-3e64-4b20-92d2-0f2faae2c626.JPG)

You can use rm to remove that file:

`sudo rm /var/www/your_domain/info.php`


## RETRIEVING DATA FROM MYSQL DATABASE WITH PHP (CONTINUED)

connect to the MySQL console using the root account:

`sudo mysql`

To create a new database, run the following command from your MySQL console:

mysql> CREATE DATABASE `example_database`;

`CREATE USER 'example_user'@'%' IDENTIFIED WITH mysql_native_password BY 'password';`
Now we need to give this user permission over the example_database database:

`GRANT ALL ON example_database.* TO 'example_user'@'%';`
This will give the example_user user full privileges over the example_database database, while preventing this user from creating or modifying other databases on your server.
![db](https://user-images.githubusercontent.com/33351638/146084985-dbc1b046-fdd3-4bed-853c-8707442acdd4.JPG)

test if the new user has the proper permissions by logging in to the MySQL console again, this time using the custom user credentials:

`mysql -u example_user -p`
`SHOW DATABASES;`

Next, we’ll create a test table named todo_list. From the MySQL console, run the following statement:

`CREATE TABLE example_database.todo_list (
  item_id INT AUTO_INCREMENT,
 content VARCHAR(255),
  PRIMARY KEY(item_id) );`
  
  Adding values to database
  `INSERT INTO example_database.todo_list (content) VALUES ("My first important item");`
  
  To see the records...
  
  `SELECT * FROM example_database.todo_list;`
  
  
  Create a new PHP file in your custom web root directory using your preferred editor. We’ll use vi for that:

`sudo nano /var/www/edumixlemp/todo_list.php`
  
 paste the code below
 
 `<?php
$user = "example_user";
$password = "password";
$database = "example_database";
$table = "todo_list";
try {
  $db = new PDO("mysql:host=localhost;dbname=$database", $user, $password);
  echo "<h2>TODO</h2><ol>";
  foreach($db->query("SELECT content FROM $table") as $row) {
    echo "<li>" . $row['content'] . "</li>";
  }
  echo "</ol>";
} catch (PDOException $e) {
    print "Error!: " . $e->getMessage() . "<br/>";
    die();
}`

![php_script](https://user-images.githubusercontent.com/33351638/146084857-9f741d36-029b-447c-83cd-32c63e85d802.JPG)
![php_scriptweb](https://user-images.githubusercontent.com/33351638/146084860-71f7e728-48f7-4b2b-b02a-6526dcb445a4.JPG)

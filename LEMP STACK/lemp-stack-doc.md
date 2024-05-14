LEMP STACK IMPLEMENTATION IN AWS

Introduction:

The LEMP stack is a popular open-source web development platform that consists of four main components: Linux, Nginx, MySQL, and PHP (or sometimes Perl or Python). This documentation outlines the setup, configuration, and usage of the LEMP stack.

Creating EC2 Instance and establishing SSH Connection.

Created an AWS Account after which a new EC2 instance of the t2.micro family within the free tier was created with an Ubuntu Server Installed. A new key pair was created and downloaded to the local machine.

![alt text](Images/ec2_ssh_access.PNG)

STEP 1 - Installing the Nginx Web Server 

 Update and Update the server's package.

sudo apt update
sudo apt upgrade -y

![alt text](Images/ec2_update.PNG)

![alt text](Images/upgrade-ec2-packages.PNG)

Install the nginx package

![alt text](Images/nginx_installed.PNG)

Run the commaand below to confirm the status of nginx.

sudo systemctl status nginx

![alt text](Images/nginx_status_check.PNG)

Access nginx locally on the Ubuntu shell

curl http://localhost:80

![alt text](Images/curl_access.PNG)

Access the Nginx web server using the public IP on port 80

![alt text](Images/nginx_default_page.PNG)

Another way to retrieve your Public IP address, other than to check it in AAWS Web console is to use the following command:

![alt text](Images/Curl_authorized.PNG)


STEP 2- Installing MySQL

Again, use 'apt' to acquire and install this software:

sudo apt install mysql-server

![alt text](Images/my_sql_server_installed.PNG)

When the installation is finished, log in to the MySQL console by typing:

$ sudo mysql

![alt text](Images/root_password_set.PNG)

Set a password for root user using mysql_native_password as default authentication method.

Here, the user's password was defined as "Pasw0rd123$"

ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'Passw0rd123$';

And type 'exit'.

After changing root user password, log in to MySQL console.

A command prompt for password was noticed after running the command below.

sudo mysql -p

![alt text](Images/mysql_console_accessed.PNG)

Exit MySQL shell

exit


STEP 3 - Install PHP Package

Install php-fpm (PHP fastCGI process manager) and tell nginx to pass PHP requests to this software for processing. Also, install php-mysql, a php module that allows PHP to communicate with MySQL-based databases. Core PHP packages will automatically be installed as dependencies.

The following were installed:

php-fpm (PHP fastCGI process manager)
php-mysql

$ sudo apt install php-fpm php-mysql -y

![alt text](Images/php_installed.PNG)

STEP 4 - Configuring Nginx to Use PHP Processor

Create the root web directory for projectLEMP domain as follows:

$ sudo mkdir /var/www/projectLEMP

Next, assign ownership of the directory with the $USER environment variaable, which will reference the current system user:

$ sudo chown -R $USER:$USER /var/www/projectLEMP

![alt text](Images/ProjectLemp_dir_created.PNG)

Create a new configuration file in Nginx’s “sites-available” directory.

$ sudo nano /etc/nginx/sites-available/projectLEMP

Paste in the following bare-bones configuration and it is important to note and modify your php version and aapply to your script:

server {
  listen 80;
  server_name projectLEMP www.projectLEMP;
  root /var/www/projectLEMP;

  index index.html index.htm index.php;

  location / {
    try_files $uri $uri/ =404;
  }

  location ~ \.php$ {
    include snippets/fastcgi-php.conf;
    fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;
  }

  location ~ /\.ht {
    deny all;
  }
}

Here’s what each directives and location blocks does:

- listen - Defines what port nginx listens on. In this case it will listen on port 80, the default port for HTTP.

- root - Defines the document root where the files served by this website are stored.

- index - Defines in which order Nginx will prioritize the index files for this website. It is a common practice to list index.html files with a higher precedence than index.php files to allow for quickly setting up a maintenance landing page for PHP applications. You can adjust these settings to better suit your application needs.

- server_name - Defines which domain name and/or IP addresses the server block should respond for. Point this directive to your domain name or public IP address.

- location / - The first location block includes the try_files directive, which checks for the existence of files or directories matching a URI request. If Nginx cannot find the appropriate result, it will return a 404 error.

- location ~ .php$ - This location handles the actual PHP processing by pointing Nginx to the fastcgi-php.conf configuration file and the php7.4-fpm.sock file, which declares what socket is associated with php-fpm.

- location ~ /.ht - The last location block deals with .htaccess files, which Nginx does not process. By adding the deny all directive, if any .htaccess files happen to find their way into the document root, they will not be served to visitors.

Activate the configuration by linking to the config file from Nginx’s sites-enabled directory

$ sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/

This will tell Nginx to use this configuration when next it is reloaded.

Test the configuration for syntax error

$ sudo nginx -t

![alt text](Images/nginx_syntax_error_check.PNG)

Disable the default Nginx host that currently configured to listen on port 80

$ sudo unlink /etc/nginx/sites-enabled/default

![alt text](Images/unlink_and_reload_of_nginx.PNG)

The new website is now active but the web root /var/www/projectLEMP is still empty. Create an index.html file in this location so to test the virtual host work as expected.

$ sudo echo ‘Hello LEMP from hostname’ $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) ‘with public IP’ $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectLEMP/index.html

![alt text](Images/index_file_created.PNG)

When setup correctly, the following should be the result when the public IP or Public DNS is viewed in a browser:

![alt text](Images/site_url_ip_access.PNG)

Open it with public dns name (port is optional)

![alt text](Images/site_url_accessed_via_dns.PNG)

STEP 5 - Test PHP with Nginx

After successfully setting up the LEMP stack, it is important to test to validate that Nginx handles .php correctly.

Create a new file called info.php within the document root in the command: nano /var/www/projectLEMP/info.php

Paste in the following in the file: <?php
phpinfo();


Access the page on the browser and attach /info.php

http://public-ip/info.php

![alt text](Images/php_page.PNG)

After checking the relevant information about the server through this page, It’s best to remove the file created as it contains sensitive information about the PHP environment and the ubuntu server. It can always be recreated if the information is needed later.

$ sudo rm /var/www/projectLEMP/info.php

STEP 6 - Retrieving data from MySQL database with PHP

Create a new user with the mysql_native_password authentication method in order to be able to connect to MySQL database from PHP.
Create a database named todo_database and a user named todo_user

First, connect to the MySQL console using the root account.

$ sudo mysql -p

Create a new database

$ CREATE DATABASE todo_database;

![alt text](Images/new_db_created.PNG)

Create a new user and grant the user full privileges on the new database.

CREATE USER 'todo_user'@'%' IDENTIFIED WITH mysql_native_password BY 'Passw0rd123$';

GRANT ALL ON todo_database.* TO 'todo_user'@'%';

![alt text](Images/user_creation_and_privilege.PNG)

Type 'exit' to exit shell.

Login to MySQL console with the user custom credentials and confirm that you have access to todo_database.

$ mysql -u todo_user -p

SHOW DATABASES;

![alt text](Images/db_details_captured.PNG)

Create a test table named todo_list.

From MySQL console, run the following:

CREATE TABLE todo_database.todo_list (
  item_id INT AUTO_INCREMENT,
  content VARCHAR(255),
  PRIMARY KEY(item_id)
);

Insert a few rows of content to the test table.

INSERT INTO todo_database.todo_list (content) VALUES ("My first important item");

INSERT INTO todo_database.todo_list (content) VALUES ("My second important item");

INSERT INTO todo_database.todo_list (content) VALUES ("My third important item");

![alt text](Images/rows_inserted.PNG)

To confirm that the data was successfully saved to the table run:

SELECT * FROM todo_database.todo_list;

![alt text](Images/query_table_details.PNG)

Type 'exit'

Create a PHP script that will connect to MySQL and query the content.

Create a new PHP file in the custom web root directory

$ sudo nano /var/www/projectLEMP/todo_list.php

Copy the content below into the todo_list.php script.

<?php
$user = "todo_user";
$password = "Admin123$";
$database = "todo_database";
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
}
?>

![alt text](Images/contents_of_php_script.PNG)

I accessed this page in the web browser by visiting the domain name or public IP address configured for the website, followed by /todo_list.php: 

http://<public_IP>/todo_list.php

![alt text](Images/php_site_ip_access.PNG)



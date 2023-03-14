# INSTALLING THE NGINX WEB SERVER

>In order to display web pages to our site visitors, we are going to employ Nginx, a high-performance web server. We’ll use the apt package manager to install this package.
Since this is our first time using apt for this session, start off by updating your server’s package index. Following that, you can use apt install to get Nginx installed:
```
sudo apt update
sudo apt install nginx
```

![installing-nginx](https://user-images.githubusercontent.com/125014455/225077272-4daefec2-d5bd-4cdd-ab70-e043f3457680.png)

---

>When prompted, enter Y to confirm that you want to install Nginx. Once the installation is finished, the Nginx web server will be active and running on your Ubuntu 20.04 server.

>To verify that nginx was successfully installed and is running as a service in Ubuntu, run:
```
sudo systemctl status nginx
```

![nginx-status](https://user-images.githubusercontent.com/125014455/225077458-79758ef3-24b0-43bb-a506-9be2711907e3.png)

---

>If it is green and running, then you did everything correctly – you have just launched your first Web Server in the Clouds!

>As we know, we have TCP port 22 open by default on our EC2 machine to access it via SSH, so we need to add a rule to EC2 configuration to open inbound connection through port 80:

>First, let us try to check how we can access it locally in our Ubuntu shell, run:
```
curl http://localhost:80
or
curl http://127.0.0.1:80
```

>These 2 commands above actually do pretty much the same – they use ‘curl’ command to request our Nginx on port 80 (actually you can even try to not specify any port – it will work anyway). The difference is that: in the first case we try to access our server via DNS name and in the second one – by IP address (in this case IP address 127.0.0.1 corresponds to DNS name ‘localhost’ and the process of converting a DNS name to IP address is called "resolution"). We will touch DNS in further lectures and projects.

>As an output you can see some strangely formatted test, do not worry, we just made sure that our Nginx web service responds to ‘curl’ command with some payload.
Now it is time for us to test how our Nginx server can respond to requests from the Internet.

>Open a web browser of your choice and try to access following url
```
http://<Public-IP-Address>:80
```

>Another way to retrieve your Public IP address, other than to check it in AWS Web console, is to use following command:
```
curl -s http://169.254.169.254/latest/meta-data/public-ipv4
```

>The URL in browser shall also work if you do not specify port number since all web browsers use port 80 by default.

![welcome-nginx](https://user-images.githubusercontent.com/125014455/225077567-ee51ba2b-9d45-489b-8fd8-ac06212fb1ca.png)

---

>If you see following page, then your web server is now correctly installed and accessible through your firewall.

---
---

# INSTALLING MYSQL

>Now that you have a web server up and running, you need to install a Database Management System (DBMS) to be able to store and manage data for your site in a relational database. MySQL is a popular relational database management system used within PHP environments, so we will use it in our project.
Again, use ‘apt’ to acquire and install this software:
$ sudo apt install mysql-server
When prompted, confirm installation by typing Y, and then ENTER.

![mysql-install](https://user-images.githubusercontent.com/125014455/225077730-701352b2-99dc-47dd-a2e0-916427841cc2.png)

---

>When the installation is finished, log in to the MySQL console by typing:
```
sudo mysql
```

>This will connect to the MySQL server as the administrative database user root, which is inferred by the use of sudo when running this command. You should see output like this:

![sudo-mysql](https://user-images.githubusercontent.com/125014455/225078566-cdacdee6-5263-4fba-bb8d-b0a7f85460d9.png)

---

>It’s recommended that you run a security script that comes pre-installed with MySQL. This script will remove some insecure default settings and lock down access to your database system. Before running the script, you will set a password for the root user, just like i've done in the image above using mysql_native_password as default authentication method. We’re defining this user’s password as #PassWord1
```
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '#Password1';
```

>Exit the MySQL shell with **exit**

>Start the interactive script by running:
```
sudo mysql_secure_installation
```

>This will ask if you want to configure the VALIDATE PASSWORD PLUGIN.

>Note: Enabling this feature is something of a judgment call. If enabled, passwords which don’t match the specified criteria will be rejected by MySQL with an error. It is safe to leave validation disabled, but you should always use strong, unique passwords for database credentials.
Answer Y for yes, or anything else to continue without enabling.

![msql-secure](https://user-images.githubusercontent.com/125014455/225078726-14b857c3-0fc5-48c4-8339-212cabb8379d.png)

---

>When you’re finished, test if you’re able to log in to the MySQL console by typing:
```
sudo mysql -p
```

>Notice the -p flag in this command, which will prompt you for the password used after changing the root user password.
To exit the MySQL console, type:
mysql> exit

![mysql-p](https://user-images.githubusercontent.com/125014455/225078939-49c687fd-bf88-4246-a354-bca198d9f79c.png)

---

>Notice that you need to provide a password to connect as the root user.

>For increased security, it’s best to have dedicated user accounts with less expansive privileges set up for every database, especially if you plan on having multiple databases hosted on your server.

>Note: At the time of this writing, the native MySQL PHP library mysqlnd doesn’t support caching_sha2_authentication, the default authentication method for MySQL 8. For that reason, when creating database users for PHP applications on MySQL 8, you’ll need to make sure they’re configured to use mysql_native_password instead. We’ll demonstrate how to do that in Step 6.

>Your MySQL server is now installed and secured. Next, we will install PHP, the final component in the LEMP stack.
---
---

# INSTALLING PHP

>You have Nginx installed to serve your content and MySQL installed to store and manage your data. Now you can install PHP to process code and generate dynamic content for the web server.

>While Apache embeds the PHP interpreter in each request, Nginx requires an external program to handle PHP processing and act as a bridge between the PHP interpreter itself and the web server. This allows for a better overall performance in most PHP-based websites, but it requires additional configuration. You’ll need to install php-fpm, which stands for “PHP fastCGI process manager”, and tell Nginx to pass PHP requests to this software for processing. Additionally, you’ll need php-mysql, a PHP module that allows PHP to communicate with MySQL-based databases. Core PHP packages will automatically be installed as dependencies.

>To install these 2 packages at once, run:
```
sudo apt install php-fpm php-mysql
```

>When prompted, type Y and press ENTER to confirm installation.

>You now have your PHP components installed. Next, you will configure Nginx to use them.

![install-php](https://user-images.githubusercontent.com/125014455/225079103-02e49940-ff40-4e7f-afa1-cb1ed145feed.png)

---
---

# CONFIGURING NGINX TO USE PHP PROCESSOR

>When using the Nginx web server, we can create server blocks (similar to virtual hosts in Apache) to encapsulate configuration details and host more than one domain on a single server. In this guide, we will use project LEMP as an example domain name.

>On Ubuntu 20.04, Nginx has one server block enabled by default and is configured to serve documents out of a directory at /var/www/html. While this works well for a single site, it can become difficult to manage if you are hosting multiple sites. Instead of modifying /var/www/html, we’ll create a directory structure within /var/www for your domain website, leaving /var/www/html in place as the default directory to be served if a client request does not match any other sites.

>Create the root web directory for your domain as follows:
```
sudo mkdir /var/www/projectLEMP
```

>Next, assign ownership of the directory with the $USER environment variable, which will reference your current system user:
```
sudo chown -R $USER:$USER /var/www/projectLEMP
```

![sudo-mkdir](https://user-images.githubusercontent.com/125014455/225079271-cf9b966c-bf38-4f34-9c16-6afb926de5e2.png)

---

>Then, open a new configuration file in Nginx’s sites-available directory using your preferred command-line editor. Here, we’ll use nano:
```
sudo nano /etc/nginx/sites-available/projectLEMP
```

>This will create a new blank file. Paste in the following bare-bones configuration:
```
#/etc/nginx/sites-available/projectLEMP

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
    	fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
 	}
 
	location ~ /\.ht {
    	deny all;
	}
 
}
```

![nano](https://user-images.githubusercontent.com/125014455/225079474-63b783de-28a3-4c06-8ade-a0f26c55a101.png)

---

>When you’re done editing, save and close the file. If you’re using nano, you can do so by typing CTRL+X and then y and ENTER to confirm.

>Activate your configuration by linking to the config file from Nginx’s sites-enabled directory:
```
sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/
```

>This will tell Nginx to use the configuration next time it is reloaded. You can test your configuration for syntax errors by typing:
```
sudo nginx -t
```

>You shall see following message:
```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

![nginx-t](https://user-images.githubusercontent.com/125014455/225079625-3a8c4dfd-7579-4ae9-8a51-d835344bd73d.png)

---

>If any errors are reported, go back to your configuration file to review its contents before continuing.

>We also need to disable default Nginx host that is currently configured to listen on port 80, for this run:
```
sudo unlink /etc/nginx/sites-enabled/default
```

>When you are ready, reload Nginx to apply the changes:
```
sudo systemctl reload nginx
```

>Your new website is now active, but the web root /var/www/projectLEMP is still empty. Create an index.html file in that location so that we can test that your new server block works as expected:
```
sudo echo 'Hello LEMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectLEMP/index.html
```

>Now go to your browser and try to open your website URL using IP address:
```
http://<Public-IP-Address>:80
```

![hello-lemp](https://user-images.githubusercontent.com/125014455/225079749-c5f90cbe-64a7-4048-a1bf-9d49c2d3588a.png)

---
---

# TESTING PHP WITH NGINX

>Your LEMP stack should now be completely set up.

>At this point, your LAMP stack is completely installed and fully operational.

>You can test it to validate that Nginx can correctly hand .php files off to your PHP processor.

>You can do this by creating a test PHP file in your document root. Open a new file called info.php within your document root in your text editor:
```
sudo nano /var/www/projectLEMP/info.php
```

>Type or paste the following lines into the new file. This is valid PHP code that will return information about your server:
```
<?php
phpinfo();
```
![nano-php](https://user-images.githubusercontent.com/125014455/225079819-491c5952-c334-471f-954e-e7ddcddd6316.png)

---

>You can now access this page in your web browser by visiting the domain name or public IP address you’ve set up in your Nginx configuration file, followed by /info.php:
```
http://`server_domain_or_IP`/info.php
```

>You will see a web page containing detailed information about your server:

![php](https://user-images.githubusercontent.com/125014455/225079889-1725ab64-7fef-489b-8682-507724438aa0.png)

---

>After checking the relevant information about your PHP server through that page, it’s best to remove the file you created as it contains sensitive information about your PHP environment and your Ubuntu server. You can use rm to remove that file:
```
sudo rm /var/www/your_domain/info.php
```

>You can always regenerate this file if you need it later.

---
---

# RETRIEVING DATA FROM MYSQL DATABASE WITH PHP

>n this step you will create a test database (DB) with simple "To do list" and configure access to it, so the Nginx website would be able to query data from the DB and display it.

>At the time of this writing, the native MySQL PHP library mysqlnd doesn’t support caching_sha2_authentication, the default authentication method for MySQL 8. We’ll need to create a new user with the mysql_native_password authentication method in order to be able to connect to the MySQL database from PHP.

>We will create a database named example_database and a user named example_user, but you can replace these names with different values.

>First, connect to the MySQL console using the root account:
```
sudo mysql
```

>To create a new database, run the following command from your MySQL console:
```
mysql> CREATE DATABASE `example_database`;
```

Now you can create a new user and grant him full privileges on the database you have just created.

>The following command creates a new user named example_user, using mysql_native_password as default authentication method. We’re defining this user’s password as password, but you should replace this value with a secure password of your own choosing.
```
mysql> CREATE USER 'example_user'@'%' IDENTIFIED WITH mysql_native_password BY 'password';
```

>Now we need to give this user permission over the example_database database:
```
mysql> GRANT ALL ON example_database.* TO 'example_user'@'%';
```

>This will give the example_user user full privileges over the example_database database, while preventing this user from creating or modifying other databases on your server.

>Now exit the MySQL shell with: **exit**

>You can test if the new user has the proper permissions by logging in to the MySQL console again, this time using the custom user credentials:
```
mysql -u example_user -p
```

>Notice the -p flag in this command, which will prompt you for the password used when creating the example_user user. After logging in to the MySQL console, confirm that you have access to the example_database database:
```
mysql> SHOW DATABASES;
```

>This will give you the following output:
```
Output
+--------------------+
| Database       	|
+--------------------+
| example_database   |
| information_schema |
+--------------------+
2 rows in set (0.000 sec)
```

![mysql-u-example](https://user-images.githubusercontent.com/125014455/225080075-52f00cce-996b-4914-931c-d6a4094c86dc.png)

---
>Next, we’ll create a test table named todo_list. From the MySQL console, run the following statement:
```
CREATE TABLE example_database.todo_list (
mysql> 	item_id INT AUTO_INCREMENT,
mysql> 	content VARCHAR(255),
mysql> 	PRIMARY KEY(item_id)
mysql> );
```

>Insert a few rows of content in the test table. You might want to repeat the next command a few times, using different VALUES:
```
mysql> INSERT INTO example_database.todo_list (content) VALUES ("My first important item");
```

>To confirm that the data was successfully saved to your table, run:
```
mysql> SELECT * FROM example_database.todo_list;
```

![select-from-example](https://user-images.githubusercontent.com/125014455/225080229-36f3c95e-d794-440a-b730-bae20ddf477e.png)

---

>After confirming that you have valid data in your test table, you can exit the MySQL console with **exit**

>Now you can create a PHP script that will connect to MySQL and query for your content. Create a new PHP file in your custom web root directory using your preferred editor. We’ll use vi for that:
```
nano /var/www/projectLEMP/todo_list.php
```

>The following PHP script connects to the MySQL database and queries for the content of the todo_list table, displays the results in a list. If there is a problem with the database connection, it will throw an exception.

>Copy this content into your todo_list.php script:
```
<?php
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
}
```

 file:///home/aoa-clothings/Documents/DEVOPS/PBL%20Project/project%202/nano-example.png

>Save and close the file when you are done editing.

>You can now access this page in your web browser by visiting the domain name or public IP address configured for your website, followed by /todo_list.php:
```
http://<Public_domain_or_IP>/todo_list.php
```
You should see a page like this, showing the content you’ve inserted in your test table:

![todo-list](https://user-images.githubusercontent.com/125014455/225080467-9616b85f-3543-4298-80b5-eaf88ad53d18.png)

---

>That means your PHP environment is ready to connect and interact with your MySQL server.# MY-PBL-PROJECT2

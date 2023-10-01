# LEMP stack implementation

## Step 0 - Preparing Prerequisites

### connecting to AWS EC2 server

i opened my git bash and changed my directory using the `cd` command using the code below

`cd Downloads`

***i have my private key file in this `Downloads` directory and that is the reason for the change***

![change directory](<images/1 change directory to private key location.PNG>)

to connect to AWS server type the code below

`ssh -i <Your-private-key.pem> ubuntu@<EC2-Public-IP-address>`

![connect to aws server](<images/2 connect to aws server instance.PNG>)

![connect to aws server 2](<images/3 connect to aws server instance.PNG>)

## Step one - Installing the Nginx Web Server 

To use the Nginx Web Server we have to install it, but first of all i updated my server to do so i use the code below

`sudo apt update`

![update the server](<images/4 update server.PNG>)

To install the Nginx Web Server i use the code below 

`sudo apt install nginx`

![install Nginx server](<images/5 install nginx server.PNG>)

To verify i have installed the Nginx successfully i typed the code below

`sudo systemctl status nginx`

![nginx server successfully installed](<images/6 verify nginx is installed on the server.PNG>)

Over to my AWS console i opened TCP port 80 in my inbound rules to be able to recieve any traffic.

![security group configured successfully](<images/7 configure security group to accept incoming traffic.PNG>)

To confirm i can access my server can i connected using `DNS` name as seen below 

`curl http://localhost:80`

![connect using dns on commnd shell](<images/8 check webpage on ubuntu shell.PNG>)

and connect using ip address i use the code below

`curl http://127.0.0.1:80
`

![connect using ip address](<images/9 check webpage on ubuntu shell.PNG>)

To see how my Nginx server responds to to request on the internet i access using my ip address on the web browser

![visit nginx on web browser](<images/10 visit nginx on webpage.PNG>)

To be able to identify my my ip address without visiting my aws console i used the code below 

`curl -s http://169.254.169.254/latest/meta-data/public-ipv4`

![know ip address on console ](<images/11 Determine the public ip of the server.PNG>)


## Step two - Installing MySQL

To install MySQL on the server i use `apt` to acquire and install the software

`sudo apt install mysql-server`

![install mysql](<images/12 install mysql DBMS.PNG>)

after installing i log into MySQL console using the code below

`sudo mysql`

![log into mysql](<images/13 log into mysql DBMS.PNG>)

By recomendation i have to run a security script to remove some insecure default settings. Running this script will secure my database. 

Before running this script i set a root password using the code below

`ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';`

![set password for mysql database](<images/14 set password for mysql.PNG>)

Afterwards i exited MySQL with the command below.

`exit`
![xit mysql database](<images/15 exit mysql dbms.PNG>)

i started my interactive script using the code below

`sudo mysql_secure_installation`

![interractive script](<images/16 interractive script.PNG>)

To test if i can log into MySQL console i typed the code below

`sudo mysql -p`

![login test into php](<images/17 test if able to login to mysql data base.PNG>) *the `-p` flag will prompt for password after changing the root password*

afterwhich i exited MySQL console using the code below

`exit`

![exit mysql](<images/18 exit mysql.PNG>)

## Step three - Installing PHP 

i will be installing two pagkages of php namely 

`php-fpm` and `php-mysql`

to install the two packages simultaneously i used the code below

`sudo apt install php-fpm php-mysql`

![install php](<images/19 install php.PNG>)

## Step four - Configuring Nginx to use PHP Processor

In this step i will use `projectLEMP` as my example domain name

i create a root web directory for my domain using the code below.

`sudo mkdir /var/www/projectLEMP`

![create directory for my domain](<images/20 create directory with projectLEMP.PNG>)

i assigned ownership of the directory with `$USER` environment variable which will refrence my current system  user

`sudo chown -R $USER:$USER /var/www/projectLEMP`

![assign ownership of the directory](<images/21 assign ownership of directory to USER.PNG>)

next i open a new configuration file in Nginx `site-available` directory using `nano` command line editor using the code below

`sudo nano /etc/nginx/sites-available/projectLEMP`

**Next i typed the below text into the text editor** 

#/etc/nginx/sites-available/projectLEMP

server {
    listen 80;
    server_name projectLEMP www.projectLEMP;
    root /var/www/projectLEMP;`

    ndex index.html index.htm index.php;

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

}`


![open a new configuration](<images/22 create new file and paste bare bone configuration.PNG>)

Afte editing i save and close the text editor.

To do this is typed `ctrl+X` and then `y` and `enter` to confirm


To activate my configuration by lilnking to the config file from Nginx's `site-enabled` directory i used the code below. 

`sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled/`

![activation of configuration file](<images/23 link the configuration file to activate it.PNG>)

doing so tells Nginx to use the configuration next time it is reloaded.

To test my configuration for syntax errors i use the code below

`sudo nginx -t`

![test for syntax errors](<images/24 text for syntax error.PNG>)


the output of the test should be 

`nginx: the configuration file /etc/nginx/nginx.conf syntax is ok`
`nginx: configuration file /etc/nginx/nginx.conf test is successful`


if any error is reported i would go back to my configuration file and review its content before continuing.


I also disabled any default Nginx host that is currently configured to listen to port 80. To to this i used the code below

`sudo unlink /etc/nginx/sites-enabled/default`

![disable listen port 80](<images/25 disable nginx that is configured to listen to port 80.PNG>)

My website is now active but the web root/var/www/projectLEMP is still empty so i created an index.html file in that location so that i can test that my new server block works as i expects. To do so i use the code below

`sudo echo 'Hello LEMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectLEMP/index.html`

next i opened my website url on my browser using my ip address using the code format below 

`http://<Public-IP-Address>:80`

![display webpage in browser](<images/28 display output in web browser using ip address.PNG>)

i also opened my webpage using its DNS name using the code format below

`http://<Public-DNS-Name>:80`

![output using dns name](<images/29 display output in web browser using DNS address.PNG>)

## Step five - Testing PHP with Nginx

To test and validate that Nginx can correctly hand `.php` files to my php processor i create a php test file in my document root and open a file named `info.php` using the code below and nano text editor

`nano /var/www/projectLEMP/info.php`

![create php test file](<images/30 create php file in document root.PNG>)

next i typed the code below in the nano code editor 

`<?php
phpinfo();`

![typed the code into the text editor](<images/31 create php file in document root.PNG>)

To access this page in my web browser i typed the code below into my web browser.

`http://"server_domain_or_IP"/info.php`

![view php in web browser](<images/32 display php webpage on browser.PNG>)

after checking the relevant information i removed the file i just created because it contains sensitive information about my php environment and my server. i used the code below to remove the file

`sudo rm /var/www/your_domain/info.php`

![removed the php file](<images/33 remove sensitive php file.PNG>)

## Step six - Retrieving data from MySQL database with PHP

First of all i connect to MySQL console using the `root` account usint the code below

`$ sudo mysql`

![log into mysql console](<images/34 log into mysql.PNG>)

next i created a new database using the code below

`mysql> CREATE DATABASE `example_database`;`

![create a database](<images/35 create new database.PNG>)

next i created a new user named `example_user` and secured it with a password `PassWord.1` using the code below

`mysql>  CREATE USER 'example_user'@'%' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';`

![create a new user and set passord for it](<images/36 create user and set password.PNG>)

next i gave this user permission to access `example_database` database file using the code below

`mysql> GRANT ALL ON example_database.* TO 'example_user'@'%';`

![grant access to new user](<images/37 grant new user access to the database.PNG>)

next i exited MySQL console usint the code below

`exit`

![mysql exit](<images/38 exit mysql.PNG>)

To confirm if the new user has access to the database i logged into MySQL as `example_user` user i created using the code below

`mysql -u example_user -p`

![log into mysql as new user](<images/39 login to mysql using the new user access.PNG>)

note `-p` flag will promp for a password used when creating the `example_user`

to confirm `example_user` has access to `example_database` i used the code below

`SHOW DATABASES;`

the desired output will be in this format

![list out database of new user](<images/40 list out the database the new user has access to.PNG>)

next i create a test table named `todo_list` to do this i run the command below

`CREATE TABLE example_database.todo_list (item_id INT AUTO_INCREMENT,content VARCHAR(255),PRIMARY KEY(item_id));`

![create table todo_list](<images/41 create table todo list.PNG>)

afterwards i added a few rows of content to the test table to do so i  use the code below multiple times

`INSERT INTO example_database.todo_list (content) VALUES ("My first important item");`

![add data to table](<images/42 add data to table.PNG>)

![add more data to table](<images/43 add more info to table.PNG>)

to confirm data was saved successfully i run the code below

`SELECT * FROM example_database.todo_list;`

The desired output is to be in the format below and comprising of the data i inputed


![confirm data is saved to table](<images/44 confirm data was saved to table and display.PNG>)

After confirming the validity of my data i exited MySQL console using the code below

`exit`

![exit mysql](<images/45 exit mysql table.PNG>)

next i created a php script that will connect to MySQL and query for my content. i created a new php file in my custom web root directory using vi editor using the code below

`vi /var/www/projectLEMP/todo_list.php`

next i typed the content in the image below into the `todo_list.php` script on the vi editor after which i save and close the editor

![type php script](<images/46 connect to php script.PNG>)

to access this page i visit it using the ip adress followed by `/todo_list.php` as the code format below

`http://<Public_domain_or_IP>/todo_list.php`

![visit php webpage](<images/47 view php data on web browser.PNG>)
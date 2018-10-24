# Project Overview

Taken a baseline installation of a Linux server and prepared it to host my web applications - Item Catalog Project. Secured the server from a number of attack vectors, installed and configured a database server, and deployed Item Catalog Project onto it.

Available at - [Link](http://ec2-54-93-222-165.eu-central-1.compute.amazonaws.com/)
Public IP - 54.93.222.165

# Get your server
## Start a new Ubuntu Linux server instance on Amazon Lightsail. 
Details on setting up your Lightsail Instance are available at [Udacity's page](https://classroom.udacity.com/nanodegrees/nd004-intr2/parts/d21e4093-10a7-4ef3-a56f-4623e05161b0/modules/e5bd1d0f-22bc-45bd-b512-4c59d7220ae9/lessons/3573679011239847/concepts/c4cbd3f2-9adb-45d4-8eaf-b5fc89cc606e)

## SSH into the server 
1. Download Private Key from the SSH keys section in the Account section on Amazon Lightsail.
2. Move the private key file into the folder ~/.ssh (where ~ is your environment's home directory). So if you downloaded the file to the 3. Downloads folder, just execute the following command in your terminal. mv ~/Downloads/Lightsail-key.pem ~/.ssh/
4. Open your terminal and type in chmod 400 ~/.ssh/Lightsail-key.pem
5. In your terminal, type in ssh -i ~/.ssh/Lightsail-key.pem ubuntu@54.93.222.165

# Secure your server
## Update all currently installed packages by 
```linux
$ sudo apt-get update
$ sudo apt-get upgrade
```
## Change the SSH port from 22 to 2200. Make sure to configure the Lightsail firewall to allow it.
1. Change port from 22 to 2200, save and quit.
```linux
$ sudo nano /etc/ssh/sshd_config
```
2. Reload SSH service using - 
```linux
$ sudo service ssh restart
```
3. Add and save port 2200 with Application as Custom and Protocol as TCP in the Networking section of your instance on Amazon Lightsail.

## Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
```linux
$ sudo ufw allow ssh
$ sudo ufw allow www
$ sudo ufw allow ntp
$ sudo ufw allow 2200/tcp
$ sudo ufw allow 80/tcp
$ sudo ufw allow 123/udp
$ sudo ufw enable 
$ sudo ufw status
```
# Give grader access
1. Create a new user named grader
```linux
$ sudo adduser grader
```
2. Give grader permission to sudo
```linux
$ sudo nano /etc/sudoers.d/grader
```
Type in grader ALL=(ALL:ALL) NOPASSWD:ALL, save and quit

3. Create an SSH key pair for grader using the ssh-keygen tool
```linux
$ ssh-keygen
```
The private key is saved in ~/.ssh on local machine, and is the one with no extension

Deploy public key on developement enviroment. 
* Public key is the one with .pub extension. View it's content with - 
```$ cat /home/ubuntu/.ssh/id_rsa.pub```
* Copy and paste this content somewhere.
* On you virtual machine:
```linux
$ su - grader
$ mkdir .ssh
$ nano .ssh/authorized_keys
```
* Copy the public key content generated on your local machine to this file and save

```
$ chmod 700 .ssh
$ chmod 644 .ssh/authorized_keys
```
* reload SSH using 
```$ service ssh restart```

* Now you can use ssh to login with the new user you created

```$ ssh -i ~/.ssh/[privateKeyFilename] grader@54.93.222.165```

# Prepare to deploy my project
## Configure the time zone, and set it to UTC
```
$ sudo dpkg-reconfigure tzdata
```
## Install and configure Apache to serve a Python mod_wsgi application.

1. Install Apache
```
$ sudo apt-get install apache2
```
2. Insatll mode_wsgi
```
$ sudo apt-get install python-setuptools libapache2-mod-wsgi
```
3. Restart apache
```
$ sudo service apache2 restart
```
## Install and configure PostgreSQL
1. Install PostgreSQL 
```$ sudo apt-get install postgresql```

2. Check if no remote connections are allowed 
```$ sudo vi /etc/postgresql/9.3/main/pg_hba.conf```

3. Login as user "postgres" 
```$ sudo su - postgres```

4. Get into postgreSQL shell 
```$ psql```

5. Create a new database named catalog and create a new user named catalog in postgreSQL shell
```postgres=# CREATE DATABASE catalog;
postgres=# CREATE USER catalog;
```
6. Set a password for user catalog
```postgres=# ALTER ROLE catalog WITH PASSWORD 'password';```

7. Give user "catalog" permission to "catalog" application database
```postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;```

8. Quit postgreSQL 
```postgres=# \q```

9. Exit from user "postgres" 
```exit```

# Deploy the Item Catalog Project
## Clone and setup your Item Catalog project from the Github repository
1. Install Git using ```$ sudo apt-get install git``` 

2. Use ```$ cd /var/www``` to move to the /var/www directory

3. Create the application directory ```$ sudo mkdir FlaskApp```

4. Move inside this directory using ```$ cd FlaskApp```

5. Clone the Item-Catalog-Project to the virtual machine ```$ git clone https://github.com/payPan22/Item-Catalog-Project.git```

6. Rename the project's name ```$ sudo mv ./catalog2 ./FlaskApp```

7. Move to the inner FlaskApp directory using ```$ cd FlaskApp```

8. Rename catalog_backend.py to __init__.py using ```$ sudo mv website.py __init__.py```, if __init__.py not present.

9. Edit database_catalog_setup.py and database_populate_items.py to change engine = create_engine('sqlite:///catalog.db') to engine = create_engine('postgresql://catalog:password@localhost/catalog'), if not already done.

10. Install pip ```$ sudo apt-get install python-pip```

11. Use pip to install dependencies -

```$ sudo pip install sqlalchemy flask-sqlalchemy psycopg2 bleach requests```

```$ sudo pip install flask packaging oauth2client redis passlib flask-httpauth```

12. Install psycopg2 ```$ sudo apt-get -qqy install postgresql python-psycopg2```

13. Create database schema ```$ sudo python database_catalog_setup.py```

14. Add items to database ```$ sudo pip install database_populate_items.py```

## Configure and Enable a New Virtual Host
1. Create FlaskApp.conf to edit: ```$ sudo nano /etc/apache2/sites-available/FlaskApp.conf```

2. Add the following lines of code to the file to configure the virtual host.
```
<VirtualHost *:80>
	ServerName XX.XXX.XXX.XX
	ServerAdmin pandy.payal2212@gmail.com
	WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
	<Directory /var/www/FlaskApp/FlaskApp/>
		Order allow,deny
		Allow from all
	</Directory>
	Alias /static /var/www/FlaskApp/FlaskApp/static
	<Directory /var/www/FlaskApp/FlaskApp/static/>
		Order allow,deny
		Allow from all
	</Directory>
	ErrorLog ${APACHE_LOG_DIR}/error.log
	LogLevel warn
	CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

3. Enable the virtual host with the following command: ```$ sudo a2ensite FlaskApp```

## Create the .wsgi File
1. Create the .wsgi File
```
$ cd /var/www/FlaskApp
$ sudo nano flaskapp.wsgi
```

2. Add the following lines of code to the flaskapp.wsgi file:
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/FlaskApp/")

from FlaskApp import app as application
application.secret_key = 'super_secret_key'
```

3. Restart Apache
```$ sudo service apache2 restart```


# References 
* [Configuring Linux Web Server Course by Udacity] (https://classroom.udacity.com/courses/ud299)
* [How To Deploy a Flask Application on an Ubuntu VPS] (https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
* [How To Secure PostgreSQL on an Ubuntu VPS] (https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)


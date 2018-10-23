# Project Overview

Taken a baseline installation of a Linux server and prepared it to host my web applications - Item Catalog Project. I have secured the server from a number of attack vectors, installed and configured a database server, and deployed Item Catalog Project onto it.

# Get your server
1. Start a new Ubuntu Linux server instance on Amazon Lightsail. Details available on setting up your Lightsail Instance are available on
2. SSH into the server 
## How to SSH into the server
1. Download Private Key from the SSH keys section in the Account section on Amazon Lightsail.
2. Move the private key file into the folder ~/.ssh (where ~ is your environment's home directory). So if you downloaded the file to the 3. Downloads folder, just execute the following command in your terminal. mv ~/Downloads/Lightsail-key.pem ~/.ssh/
4. Open your terminal and type in chmod 400 ~/.ssh/Lightsail-key.pem
5. In your terminal, type in ssh -i ~/.ssh/Lightsail-key.pem ubuntu@xx.xxx.xxx.xxx

# Secure your server
## Update all currently installed packages by 
```linux
sudo apt-get update
sudo apt-get upgrade
```
## Change the SSH port from 22 to 2200. Make sure to configure the Lightsail firewall to allow it.
1. Change port from 22 to 2200, save and quit.
```linux
sudo vi /etc/ssh/sshd_config
```
2. Reload SSH service using - 
```linux
sudo service ssh restart
```
3. Add and save port 2200 with Application as Custom and Protocol as TCP in the Networking section of your instance on Amazon Lightsail.

## Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
```linux
sudo ufw allow ssh
sudo ufw allow www
sudo ufw allow ntp
sudo ufw allow 2200/tcp
sudo ufw allow 80/tcp
sudo ufw allow 123/udp
sudo ufw enable 
sudo ufw status
```
# Give grader access
1. Create a new user named grader
```linux
sudo adduser grader
```
2. Give grader permission to sudo
```linux
sudo vi /etc/sudoers
sudo touch /etc/sudoers.d/grader
sudo vi /etc/sudoers.d/grader 
```
Type in grader ALL=(ALL:ALL) NOPASSWD:ALL, save and quit

3. Create an SSH key pair for grader using the ssh-keygen tool
```linux
ssh-keygen
```
Save the private key in ~/.ssh on local machine
Deploy public key on developement enviroment

On you virtual machine:
```linux
$ su - grader
$ mkdir .ssh
$ touch .ssh/authorized_keys
$ vi .ssh/authorized_keys
```
Copy the public key (one with the extension .pub) generated on your local machine to this file and save

```
$ chmod 700 .ssh
$ chmod 644 .ssh/authorized_keys
```
reload SSH using 
```service ssh restart```

Now you can use ssh to login with the new user you created

```ssh -i ~/.ssh/[privateKeyFilename] grader@xx.xxx.xxx.xxx```

# Prepare to deploy my project
# Deploy the Item Catalog Project

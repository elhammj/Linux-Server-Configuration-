# Linux Server Configuration 

## Objective: 
A baseline installation of a Linux server using [Amazon Lightsail](https://lightsail.aws.amazon.com/ls/webapp/home/instances) and prepare it to host [EDotTech Shop](https://github.com/elhammj/EDotTech) web applications.

## Overview:
This project emphasizes how to access, secure, perform the initial configuration of a bare-bones Linux server and how to install and configure a web and database server and actually host a web application.

## How to start?
1. Sign up [Amazon Lightsail](https://lightsail.aws.amazon.com/ls/webapp/)
2. Create Ubuntu Linux server instance by choosing:
	* OS Only
	* Ubuntu 16.04 LTS 
3. Choose your instance plan:
	* The lowest membership is enough for this project
4. Give your instance a hostname
5. Wait for it to start up
6. Once it runs:
	* Copy your public key

## Follow the following steps to configure your instance:
1. Launch Virtual Machine and SSH into the Server
	* In your local machine:
		* Download the private key form Networking part which starts with "LightsailDefaultKey" and with ".pem" extention, rename the file to "Lightsail_Key.rsa " and save it in `~/.ssh/`
		* Change the permission: `chmod 600 ~/.ssh/Lightsail_key.rsa`
		* SSH to your server by using Ubuntu User `ssh -i ~/.ssh/Lightsail_Key.rsa -p 2200 ubuntu@54.93.239.136` , note that *54.93.239.136* is the public key that oyu can find it in your instance page

## Update all currently installed packages
1. Update packages: `sudo apt-get update`
2. Upgrade packages: `sudo apt-get upgrade`

## Change the SSH port from 22 to 2200
1. Change the port to 2200 in sshd_config `sudo nano /etc/ssh/sshd_config`
2. Configure the Uncomplicated Firewall (UFW), only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123), by performing the following steps:
	* sudo ufw allow 2200/tcp
	* sudo ufw allow 80/tcp
	* sudo ufw allow 123/tcp
	* sudo ufw enable
3. In Amazon Lightsail, under "Networking" section there is "Firewall" subsection:
	* Disable port range 20
	* Add HTTP - TCP - port range 80
	* Add Cusotm - UDP - port range 123
	* Add Custom - TCP - port range 2200 

**Warning:** When changing the SSH port, make sure that the firewall is open for port 2200 first, so that you don't lock yourself out of the server. When you change the SSH port, the Lightsail instance will no longer be accessible through the web app 'Connect using SSH' button. The button assumes the default port is being used. There are instructions on the same page for connecting from your terminal to the instance. Connect using those instructions and then follow the rest of the steps.

## Create "Grader" user 
1. Create grader user: `sudo adduser grader`
2. Give grader the permission to sudo: in `sudo nano /etc/sudoers.d/grader` add this line `grader ALL=(ALL:ALL) ALL`
3. Create SSH login for grader
	* Use `ssh-keygen` to generate public and private keys in your local machine, choose "grader_key" as a name for this key
	* cd to `~/.ssh`
	* Change the permission: `chmod 644 ~/.ssh/grader_key`
	* Copy the public key from your local machine to the virtual machine: `cat ~/.ssh/grader_key`
	* In Virtual Machine, create .ssh folder: `mkdir .ssh`
	* `touch .ssh/authorized_keys`
	* Paste the public key in `sudo nano .ssh/authorized_keys` in your virtual machine
	* Change the permission: `chmod 700 .ssh` and `chmod 644 .ssh/authorized_keys`
4. Reload SSH using: `service ssh restart`
5. Now you can login by using: `ssh -i ~/.ssh/grader_key -p 2200 grader@54.93.239.136`

## Prepare to deploy your project

### Configure the local timezone to UTC 
1. `sudo dpkg-reconfigure tzdata` choose "None of the above" then UTC

### Configure Apache
1. Install Apache: `sudo apt-get install apache2`
2. Install mod_wsgi: `sudo apt-get install python-setuptools libapache2-mod-wsgi`
3. Restart Apache: `sudo service apache2 restart`

### Install PostgreSQL
1. Install python2: `sudo apt-get python2`
2. Install and configure PostgreSQL: `sudo apt-get install postgresql`
3. Disable remote connections: `sudo nano /etc/postgresql/9.5/main/pg_hba.conf`
4. Catalog Database:
	* Login as postgres user: `sudo su - postgres`
	* Start psql: `psql`
	* Create a new database "catalog": `CREATE DATABASE catalog;` and a new user "catalog": `CREATE USER catalog;`
	* Set a password for user catalog: `ALTER ROLE catalog WITH PASSWORD 'catalog';`
	* Give a catalog user a role create database: `ALTER ROLE catalog CREATEDB;`

### Install git 
1. Install git: `sudo apt-get install git`
2. Clone the project respository to: `/var/www` by using `git clone https://github.com/elhammj/EDotTech`
3. Rename the application.py file: `sudo mv applicaiton.py __init__.py`
4. In __init__.py: change `app.run(host='0.0.0.0', port=5000)` to `app.run()`
5. Change `database_setup.py`, `__init__.py` and `technologyLists.py` database: 
	* `engine = create_engine('postgresql://catalog:catalog@localhost/catalog')`
6. Install psycopg2: `sudo apt-get -qqy install postgresql python-psycopg2`

### Create Flask App
1. Create catalog.conf "Flask App": `sudo nano /etc/apache2/sites-available/catalog.conf`, add the following lines:
`<VirtualHost *:80>
        ServerName 54.93.239.136
        DocumentRoot /var/www/catalog
        ServerAdmin elham.m.j@gmail.com
        WSGIScriptAlias / /var/www/catalog.wsgi
        WSGIDaemonProcess catalog python-path=/var/www/:/var/www/catalog/venv/lib/python2.7/site-packages
        WSGIProcessGroup catalog
        <Directory /var/www/>
                Order allow,deny
                Allow from all
        </Directory>
        Alias /static /var/www/catalog/static
        <Directory /var/www/catalog/static/>
                Order allow,deny
                Allow from all
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>`
2. Enable the virtual host: `sudo a2ensite catalog`

### Create wsgi File
1. Create the .wsgi File in `/var/www/` by typing: `sudo nano catalog.wsgi`
2. Add the following lines:
`#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/")
from catalog import app as application
application.secret_key = "Your_google_secret_key"`
3. Reastart Apache: `sudo service apache2 restart`

### Update Google Configuration
1. Add a new domain to oAutho consent scree: `xip.io`
2. Add authorized Javascript origins: `http://54.93.239.136.xip.io`
3. Add authorised redirect URIs: `http://54.93.239.136.xip.io/oauth2callback` and `http://54.93.239.136.xip.io/login`
4. Downlaod the updated JSON file copy the text and past it into: `sudo nano client_secrets.json`

### Open your website and enjoy using it
1. Access the app through [http://54.93.239.136.xip.io](http://54.93.239.136.xip.io)

## References
1. [Flask Application in Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
2. [WSGI](https://devops.profitbricks.com/tutorials/install-and-configure-mod_wsgi-on-ubuntu-1604-1/)
3. [SSH Login](https://www.digitalocean.com/community/tutorials/how-to-use-ssh-to-connect-to-a-remote-server-in-ubuntu)


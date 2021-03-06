# udacity-fsnd-p5 Linux-Server-Configuration

This is the fifth project for "Full Stack Web Developer Nanodegree" on Udacity. 

In this project, a Linux virtual machine needs to be configurated to support the Item Catalog website.

You can visit http://avidalh.noip.me for review the deployed website.

## Tasks
0. Launch your Virtual Machine with your Udacity account
1. Configure a new host at https://www.noip.com
2. Follow the instructions provided to SSH into your server
3. Create a new user named grader
4. Give the grader the permission to sudo
5. Update all currently installed packages
6. Change the SSH port from 22 to 2200
7. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
8. Configure the local timezone to UTC
9. Install and configure Apache to serve a Python mod_wsgi application
10. Install, setup, launch virtual environment
10. Install and configure PostgreSQL:
	- Do not allow remote connections
	- Create a new user named catalog that has limited permissions to your catalog application database
11. Install git, clone and setup your Catalog App project (from your GitHub repository from earlier in the Nanodegree program) so that it functions correctly when visiting your server’s IP address in a browser. Remember to set this up appropriately so that your .git directory is not publicly accessible via a browser!
12. Install and configure munin
13. Install and configure fail2ban
14. Install and configure unnatended-upgrades

----

## 0. Launch Virtual Machine
The Virtual Machine was launched by following the instructions at https://www.udacity.com/account#!/development_environment.

## 1. Configure a new host at https://www.noip.com
In order to get a hostname.domainname in our machine we can sign up in any free service as www.no-ip.com, and register our IP with the desired hostname and any of the free domainnames avaliable.
In my case the I've associated the Udacity provided IP: **54.186.70.167** to the HOSTNAME.DOMAINNAME **avidalh.noip.com**.

## 2. Instructions for SSH access to the instance
1. Download Private Key provided by Udacity NanoDegree
2. Move the private key file into the folder `~/.ssh` (where ~ is your environment's home directory). So if you downloaded the file to the Downloads folder, just execute the following command in your terminal.
	```mv ~/Downloads/udacity_key.rsa ~/.ssh/```
3. Open your terminal and type in
	```chmod 600 ~/.ssh/udacity_key.rsa```
4. In your terminal, type in
	```ssh -i ~/.ssh/udacity_key.rsa root@avidalh.noip.me```

5. Development Environment Information
	Public IP Address: 54.186.70.167
	Site hostname.domainname: avidalh.noip.me
	Private Key ( is not provided here for security reasons. )

## 3. Create a new user named grader
	`sudo adduser grader`

## 4. Give the grader the permission to sudo
	`gpasswd -a grader sudo`

### Set ssh login using keys
1. generate keys on local machine using`ssh-keygen` ; then save the private key in `~/.ssh` on local machine
2. deploy public key on developement enviroment

	On you virtual machine:
	```
	$ su - grader
	$ mkdir .ssh
	$ touch .ssh/authorized_keys
	$ vim .ssh/authorized_keys
	```
	Copy the public key generated on your local machine to this file and save
	```
	$ chmod 700 .ssh
	$ chmod 644 .ssh/authorized_keys
	```
	
3. reload SSH using `service ssh restart`
4. now you can use ssh to login with the new user you created

	`ssh -i [privateKeyFilename] grader@avidalh.noip.me`

## 5. Update all currently installed packages

	sudo apt-get update
	sudo apt-get upgrade

## 6. Change the SSH port from 22 to 2200
1. Use `sudo vim /etc/ssh/sshd_config` and then change Port 22 to Port 2200 , save & quit.
2. Reload SSH using `sudo service ssh restart`

## 7. Configure the Uncomplicated Firewall (UFW)

Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)

```
	sudo ufw allow 2200/tcp
	sudo ufw allow 80/tcp
	sudo ufw allow 123/udp
	sudo ufw enable 
```

## 8. Configure the local timezone to UTC
1. Configure the time zone `sudo dpkg-reconfigure tzdata`
2. It is already set to UTC.

## 9. Install and configure Apache to serve a Python mod_wsgi application
1. Install Apache `sudo apt-get install apache2`
2. Install mod_wsgi `sudo apt-get install python-setuptools libapache2-mod-wsgi`
3. Restart Apache `sudo service apache2 restart`

## 10. Install and configure PostgreSQL
1. Install PostgreSQL `sudo apt-get install postgresql`
2. Check if no remote connections are allowed `sudo vim /etc/postgresql/9.3/main/pg_hba.conf`
3. Login as user "postgres" `sudo su - postgres`
4. Get into postgreSQL shell `psql`
5. Create a new database named catalog  and create a new user named catalog in postgreSQL shell
	
	```
	postgres=# CREATE DATABASE catalog;
	postgres=# CREATE USER catalog;
	```
	
5. Set a password for user catalog
	
	```
	postgres=# ALTER ROLE catalog WITH PASSWORD 'catalog';
	```
	
6. Give user "catalog" permission to "catalog" application database
	
	```
	postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
	```
	
7. Quit postgreSQL `postgres=# \q`
8. Exit from user "postgres" 
	
	```
	exit
	```
 
## 11. Install git, clone and setup your Catalog App project.
1. Install Git using `sudo apt-get install git`
2. Use `cd /var/www` to move to the /var/www directory 
3. Create the application directory `sudo mkdir itemCatalog`
4. Move inside this directory using `cd itemCatalog`
5. Clone the itemCatalog App to the virtual machine `git clone https://github.com/avidalh/udacity-fsnd-p3.git`
6. Rename the project's name `sudo mv ./udacity-fsnd-p3 ./itemCatalog`
7. Move to the inner FlaskApp directory using `cd itemCatalog`
8. Create python virtual enviroment

```
	cd catalog
	sudo virtualenv env
	source env/bin/activate
```

9. Rename `application.py` to `__init__.py` using `sudo mv application.py __init__.py`
10. Edit `database_setup.py`, `__init__.py` and `populate.py`:

changing	`engine = create_engine('sqlite:///catalog.db')` 
to		`engine = create_engine('postgresql://catalog:catalog@localhost/catalog')`

11. Install pip `sudo apt-get install python-pip`
12. Use pip to install dependencies `sudo pip install -r requirements.txt`
13. Install psycopg2 `sudo apt-get -qqy install postgresql python-psycopg2`
14. Create database schema `sudo python database_setup.py`
15. Check if the application works correctly

	`python __init__.py`
	
	Ctrl+c to exit if everything is ok
	
	exit virtual enviroment
	
	`deactivate`

### Configure and Enable a New Virtual Host
1. Create FlaskApp.conf to edit: `sudo nano /etc/apache2/sites-available/itemCatalog.conf`
2. Add the following lines of code to the file to configure the virtual host. 

	```
	<VirtualHost *:80>
		ServerName avidalh.noip.me
		ServerAdmin admin@mywebsite.com
		WSGIScriptAlias / /var/www/itemCatalog/itemCatalog.wsgi
		<Directory /var/www/itemCatalog/itemCatalog/>
			Order allow,deny
			Allow from all
		</Directory>
		Alias /static /var/www/itemCatalog/itemCatalog/static
		<Directory /var/www/itemCatalog/itemCatalog/static/>
			Order allow,deny
			Allow from all
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
	</VirtualHost>
	```

3. Enable the virtual host with the following command: `sudo a2ensite itemCatalog`

### Create the .wsgi File
1. Create the .wsgi File under /var/www/itemCatalog: 
	
	```
	cd /var/www/itemCatalog
	sudo nano itemCatalog.wsgi 
	```
	
2. Add the following lines of code to the flaskapp.wsgi file:
	
	```
	#!/usr/bin/python
	import sys
	import logging
	logging.basicConfig(stream=sys.stderr)
	sys.path.insert(0,"/var/www/itemCatalog/")

	from itemCatalog import app as application
	application.secret_key = 'super_secret_key'
	```

### Restart Apache
Restart Apache `sudo service apache2 restart `
At this moment we could load the site in any web browser at **http://avidalh.noip.me**


## 12. Install and configure Munin
Once everything is its place and correctly working, I've decided install a system monitoring to supervise the linux box from any place using a browser. I've opted to use [Munin](http://munin-monitoring.org/) whis is an awesome monitoring package.

The installation was relatively easy by following the package documentation and a great tuto at digital ocean. Here are the links:

- http://munin-monitoring.org/wiki/Documentation
- https://www.digitalocean.com/community/tutorials/how-to-install-munin-on-an-ubuntu-vps

The monitoring systems web page is at http://avidalh.noip.me/munin/


## 13. Install and configure fail2ban
To get safer system protected against repetitive ssh log in atempts I've installed fail2ban package.
[**fail2ban**](http://www.fail2ban.org/) can automatically alter the iptables rules after a predefined number of unsuccessful login attempts.

To install the package I've followed the package documentation and a fantastic tutorial at digital ocean.

- http://www.fail2ban.org/wiki/index.php/MANUAL_0_8
- https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-14-04


## 14. Install and configure unnatended-upgrades
In order to get an unattended updated system the package unattended-upgrades was installed and configured to update the important packages every day. To successfully install and configure the package I followed the instructions from here:

- https://wiki.debian.org/UnattendedUpgrades
- https://help.ubuntu.com/14.04/serverguide/automatic-updates.html

To check the updates are properly updated we can check the apt-get log at `/var/log/apt/history.log`.

## References:

- https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
- https://help.ubuntu.com/14.04/serverguide/automatic-updates.html
- https://wiki.debian.org/UnattendedUpgrades
- http://www.fail2ban.org/wiki/index.php/MANUAL_0_8
- https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-14-04
- http://munin-monitoring.org/wiki/Documentation
- https://www.digitalocean.com/community/tutorials/how-to-install-munin-on-an-ubuntu-vps



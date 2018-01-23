# Linux Server Configuration
Project for Udacity's Full Stack Nanodegree.

A server to host my [Item Catalog](https://github.com/reuben777/item_category) project

Site has been deployed at http://18.218.244.46

## How to log in
1. Download private key
2. Using your favorite terminal/shell, ssh into site
	`ssh -i {path to download_key}/{private key}.pem ubuntu@18.218.244.46 -p 2200`

## Update system
	sudo apt-get update
	sudo apt-get upgrade

## Configure the local timezone to UTC
	1. Change time zone `sudo dpkg-reconfigure tzdata`
	2. Select `None of the above`
	3. Then choose `UTC`

## Change the SSH port from 22 to 2200
1. `sudo nano /etc/ssh/sshd_config`
2. Alter port `22` to `2200`
3. `sudo service ssh restart`
4. Update lightsail firewall settings
```
Application: Custom
Protocol: TCP
Port Range: 2200
```

## Server Firewall setup

	1. `sudo ufw default deny incoming`
	2. `sudo ufw default allow outgoing`
	3. `sudo ufw allow 2200/tcp`
	4. `sudo ufw allow 80/tcp`
	5. `sudo ufw allow 123/udp`
	6. `sudo ufw enable`

## Create sudo user named grader
1. `sudo adduser grader`
2. `sudo usermod -aG sudo grader`
3. `sudo visudo -f /etc/sudoers.d/grader`
4. Add the following to file:
```
%grader ALL=(ALL:ALL) NOPASSWD: ALL
```

## Set SSH Public Key
1. Generate keys using `ssh-keygen`
2. Copy Public Key
3. Create a new directory called .ssh and restrict its permissions with the following commands:
	```
	$ mkdir ~/.ssh
	$ chmod 700 ~/.ssh
	```
	Insert public key into file:
	```
	$ nano ~/.ssh/authorized_keys
	```
	Save end Exit.
4. restrict permissions `chmod 600 ~/.ssh/authorized_keys`
3. reload SSH `service ssh restart`
4. You can now ssh in with your newly created super user!

	`ssh -i {path to download_key}/{grader private key}.pem grader@18.218.244.46 -p 2200`


## Apache and mod_wsgi
1. Install Apache `sudo apt-get install apache2`
2. Install mod_wsgi `sudo apt-get install python-setuptools libapache2-mod-wsgi`
3. Restart Apache for changes to take effect `sudo service apache2 restart`

## PostgreSQL
1. Install PostgreSQL `sudo apt-get install postgresql`
2. Check if no external incoming connections are allowed `sudo vim /etc/postgresql/9.3/main/pg_hba.conf`
3. Login as "postgres" `sudo su - postgres`
4. Go to postgreSQL command line `psql`
5. Create catalog database and catalog user.

	```
	CREATE DATABASE catalog;
	CREATE USER catalog;
	ALTER ROLE catalog WITH PASSWORD 'password';
	GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
	```
7. Exit postgreSQL `\q`
8. Back to ubuntu user `exit`


## Git and Item Catelog Project setup
1. Install Git `sudo apt-get install git`
2. Go to web application directory `cd /var/www`
3. Create the Item Catalog directory `sudo mkdir Item Catalog`
4. Enter directory `cd ItemCatalog`
5. Clone Item Catalog project `git clone https://github.com/reuben777/item_category`
6. Move inner content out `sudo mv ./ItemCatalog/vagrant/catalog/ ./ItemCatalog`
7. Go to the project base root `cd FlaskApp`. Yes this is ItemCatalog/ItemCatalog
8. Rename `application.py` to `__init__.py` using `sudo mv application.py __init__.py`. This is needed for wsgi to pickup the initialization file.
9. Update `application_setup.py` and `db_populate.py` with
`sudo nano application_setup.py`
and `sudo nano db_populate.py`

Change
```
engine = create_engine('sqlite:///catalog.db')
```
to
```
engine = create_engine('postgresql://catalog:123@localhost/catalog')
```
10. Install the following dependencies:
```
sudo apt-get install python-pip
sudo apt-get -qqy install postgresql python-psycopg2
sudo apt-get install python-sqlalchemy
sudo apt-get install python-pip python-dev
sudo pip install flask
sudo pip install jinja2
sudo pip install sqlalchemy
sudo pip install requests
sudo pip install oauth2client
sudo pip install passlib
sudo pip install itsdangerous
sudo pip install flask-httpauth
```
13. Create database `sudo python database_setup.py`

## Create the .wsgi initialization file
1. Create file:

	```
	cd /var/www/ItemCatalog
	sudo nano itemcatalog.wsgi
	```
2. Add the following lines of code to the itemcatalog.wsgi file:

	```
	#!/usr/bin/python
	import sys
	import logging
	logging.basicConfig(stream=sys.stderr)
	sys.path.insert(0,"/var/www/ItemCatalog/")

	from ItemCatalog import app as application
	application.secret_key = ''super_secret_key''
	```

## Configure and Enable Hosting
1. Create and edit configuration file ItemCatalog.conf: `sudo nano /etc/apache2/sites-available/ItemCatalog.conf`

2. Configure Hosting:
	```
	<VirtualHost *:80>
		ServerName 18.218.244.46
		ServerAdmin reubengroenewald@gmail.com
		WSGIScriptAlias / /var/www/ItemCatalog/itemcatalog.wsgi
		<Directory /var/www/ItemCatalog/ItemCatalog/>
			Order allow,deny
			Allow from all
		</Directory>
		Alias /static /var/www/ItemCatalog/ItemCatalog/static
		<Directory /var/www/ItemCatalog/ItemCatalog/static/>
			Order allow,deny
			Allow from all
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
	</VirtualHost>
	```
3. Then: `sudo a2ensite FlaskApp` to enable hosting


## Restart Apache
1. Restart Apache `sudo service apache2 restart `

And then you have your application running!!! :D :D :D

## Sources:
https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-16-04
https://www.digitalocean.com/community/tutorials/how-to-edit-the-sudoers-file-on-ubuntu-and-centos
https://askubuntu.com/questions/147241/execute-sudo-without-password
https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps

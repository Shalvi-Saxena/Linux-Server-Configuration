## Deploy Python App on Linux Server

### IP Address 35.156.49.66

[35.156.49.66](http://35.156.49.66/)

### SSH Port: 2200

### Publicly available URL


## Project Details

* Updating all packages on the server
	- sudo apt-get update 
	- sudo apt-get upgrade

* Update timezone
	- sudo dpkg-reconfigure tzdata

* Create a new user account named *grader*
	- sudo adduser grader
	- Install finger
		- sudo apt-get install finger
		- *finger grader*

* Give grader access
	- *usermod -aG grader sudo*

* Secure your server
		- configure sshd_config file to change the port from 22 to 2200
		- sudo nano /etc/ssh/sshd_config
		- Change PermitRootLogin from prohibited-password to no
		- sudo service ssh restart
		- Add 2200 to the available port in lightsail

	- Configure *Firewall*
			- sudo ufw status
			- sudo ufw default deny incoming
			- sudo ufw default allow outgoing
			- sudo ufw allow 2200/tcp
			- sudo ufw allow 80/tcp
			- sudo ufw allow 123/udp
			- sudo ufw enable
			
	- Download the key-pair for your instance from lightsail.
	- Then grant it the specific permissions
		- *chmod 600 key-pair.pem*
	- To sort out the initial login issue
		- sudo nano /etc/ssh/sshd_config
		- set Password Authentication to "yes"
		- sudo service ssh restart
	- ssh grader@35.156.49.66 -p 2200 -i *key-pair.pem*

* Give grader access
	- Give grader the permission to sudo
		- sudo usermod -aG sudo grader

	- Create an SSH key pair for grader using the ssh-keygen tool.
		- On your local machine generate SSH key pair: ssh-keygen
		- login into grader account 
			- *ssh grader@35.156.49.66 -p 2200 -i *key-pair.pem*
			- Make .ssh directory
				- mkdir .ssh
				- make file to store key: *touch .ssh/authorized_keys*
		- on your local machine, read contents of the public key
		- copy the key and paste in the file you just created in *grader* : nano .ssh/authorized_keys
		- Save file
		- Set permissions of the .ssh directory and the authorized_keys file
				```sh
				- sudo chmod 700 .ssh
				- sudo chmod 644 .ssh/authorized_keys
				```
				- Set Password Authentication to "no" 

	- Login using key
		- `ssh grader@Public-IP-Address* -p 2200 -i ~/.ssh/keygen`

	- Install git.
		- *sudo apt-get install git*

	- Install and configure Apache to serve a Python mod_wsgi application.
		- sudo apt-get install apache2
		- sudo apt-get install libapache2-mod-wsgi
		- Install python-dev package: *sudo apt-get install python-dev*
		- Verify wsgi is enabled: *sudo a2enmod wsgi*

* Deploy the Item Catalog project.

	- Clone Item Catalog project
		- cd /var/www
		- sudo mkdir catalog
		- git clone https://github.com/<your_project_path>/ catalog
		- sudo *touch catalog.wsgi*
			- Paste the following in catalog.wsgi file:
			```js
				#!/usr/bin/python
				import sys
				import logging
				logging.basicConfig(stream=sys.stderr)
				sys.path.insert(0,"/var/www/catalog/")

				from catalog import app as application
				application.secret_key = 'Add your secret key'
			```

	- Install all the dependencies for Item Catalog project
		```sh
		- sudo apt-get -qqy install postgresql python-psycopg2
		- sudo apt-get install python-pip
		- sudo pip install requests
		- sudo pip install oauth2client
		- sudo pip install Flask
		- sudo pip install htpplib2
		- sudo apt-get -qqy install python-sqlalchemy
		- pip install werkzeug==0.8.3
		- pip install flask==0.9
		- pip install Flask-Login==0.1.3
		```

	- Configure & Enable New Virtual Host
		- *sudo nano /etc/apache2/sites-available/catalog.conf*
		- And paste:
			- ```js
				<VirtualHost *:80>
				  ServerName 35.156.49.66
				  ServerAdmin grader@35.156.49.66
				  WSGIScriptAlias / /var/www/catalog/catalog.wsgi
				  <Directory /var/www/catalog/catalog/>
				      Order allow,deny
				      Allow from all
				  </Directory>
				  Alias /static /var/www/catalog/catalog/static
				  <Directory /var/www/catalog/catalog/static/>
				      Order allow,deny
				      Allow from all
				  </Directory>
				  ErrorLog ${APACHE_LOG_DIR}/error.log
					  LogLevel warn
				  CustomLog ${APACHE_LOG_DIR}/access.log combined
				</VirtualHost>
			```
		- save the file
		- Enable *sudo a2ensite catalog*

	- `sudo service apache2 restart`

	- Install and setup PostgreSQL
		- sudo apt-get install postgresql
		- sudo -i -u postgres
		- psql
			- *CREATE USER catalog WITH PASSWORD 'catalog';*
			- *ALTER USER catalog CREATEDB;*
			- *CREATE DATABASE catalog WITH OWNER catalog;*
			- *\c catalog*
			- *REVOKE ALL ON SCHEMA public FROM public;*
			- *GRANT ALL ON SCHEMA public TO catalog;*
		- exit

	- Edit and config database_setup.py
		```js
		- sudo nano database_setup.py
		- engine = create_engine('postgresql://catalog:catalog@localhost/catalog')
		- sudo nano populate_databse.py
		- engine = create_engine('postgresql://catalog:catalog@localhost/catalog')
		- sudo nano __init__.py
		- engine = create_engine('postgresql://catalog:catalog@localhost/catalog')
		```
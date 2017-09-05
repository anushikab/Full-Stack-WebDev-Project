# Full-Stack-WebDev-Project
Access Details

IP address = 54.218.99.153
SSH Port = 2200
URL = http://54.218.99.153/

summary of software installed

flask -> Flask is a microframework for Python 
flask-httpauth -> It is a simple extension that simplifies the use of HTTP authentication with Flask routes.
flask-sqlalchemy -> It is a extension for Flask that adds support for SQLAlchemy to your application
Apache2 -> Web ServerAdmin
ibapache2-mod-wsgi -> Python WSGI adapter module for Apache
python -> High-level programming language containing 
python-pip -> package management system used to install and manage software packages written in Python
python-dev -> python-dev contains the header files you need to build Python extensions
packaging -> directory of Python module
oauth2client -> The oauth2client library is included with the Google APIs Client Library for Python
passlib -> Passlib is a password hashing library for Python 2 & 3 
postgresql -> open source relational database management system 
sqlalchemy -> SQLAlchemy is an open-source Python Database toolkit 
psycopg2 -> It is a DB API 2.0 compliant PostgreSQL driver 
requests -> Requests is an Apache2 Licensed HTTP library


summary of configurations made

1. Start a new Ubuntu Linux server instance on Amazon Lightsail.
2. Update all currently installed packages.
		sudo apt-get update
		sudo apt-get upgrade
3. Change the SSH port from 22 to 2200 and configure the Lightsail firewall to allow it
		sudo vim /etc/ssh/sshd_config
		change port 22 to 2200
		add custom port 2200 on Amazon Lightsail firewall
4. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
	 add firewall rule as fallow
		sudo ufw allow 2200
		sudo ufw allow 80
		sudo ufw allow 123
		sudo ufw enable
	 
5. Create a new user account named grader.
		sudo adduser grader
6. Granting sudo privilages to user grader.
		visudo -f /etc/sudoers.d/grader
		grader ALL=(ALL) NOPASSWD:ALL

7. Configure SSH Key-Based Authentication for user grader 
			ssh-keygen -t rsa
		#this will genarate id_rsa and id_rsa.pub file inside of /home/user/.ssh directory
		#Embed public with grader user
			mkdir /home/grader/.ssh
		cat ~/.ssh/id_rsa.pub /home/grader/.ssh/authorized_keys
		#change the owership and permissions
			chown -R grader:grader /home/grader/.ssh/
			chmod 700 /home/grader/.ssh
			chmod 644 /home/grader/.ssh/authorized_keys
8. Install and Secure postgreSQL 
	1.Installing packages
			sudo apt-get install postgresql postgresql-contrib
	2.cheack remort connections
			vim /etc/postgresql/9.5/main/pg_hba.conf
	3.Create new Role catalog and database named catalog
			sudo adduser catalog
		
		#Access to the default postgreSQL role
			sudo -i -u postgres
		#connecting to system
			psql
		#create role with password
			CREATE ROLE catalog WITH PASSWORD 'password';
	4.Granting limited privilages of CREATEDB and LOGIN for role
			ALTER ROLE catalog CREATEDB;
			ALTER ROLE catalog LOGIN; 
	5.Create a catalog database with the ownership of catalog user.
			CREATE DATABASE catalog OWNER catalog;
		#Accessing catalog database
			\c catalog
		#Revoke all rights
			REVOKE ALL ON SCHEMA public FROM public;
		#Grant access to the catalog role
			GRANT ALL ON SCHEMA public TO catalog;
	6.Setup database for Application
		#add sqlalchemy database engin as fallow on database_setup.py and dummybooks.py
			engine = create_engine('postgresql://catalog:password@localhost:5432/catalog');
		#Run both database_setup.py and dummybooks.py
			python database_setup.py
			python dummybooks.py
			
9. Install and configure Apache to serve a Python mod_wsgi application

			sudo apt-get install apache2 ibapache2-mod-wsgi python-dev
		1. To enable mod_wsgi, run the following command:
			sudo a2enmod wsgi 
		2. Create the directory for App
			mkdir -p /var/www/FlaskApp/FlaskApp/
		3. Clone and setup the Item catalog project from Github
		4. Configure and Enable a new Virtual Host 
			sudo vim /etc/apache2/sites-available/FlaskApp.conf	
			
				<VirtualHost *:80>
                ServerName 54.218.99.153
                ServerAdmin anushikab@gmail.com

                WSGIDaemonProcess main home=/var/www/FlaskApp/FlaskApp/
                WSGIScriptAlias / /var/www/FlaskApp/FlaskApp/main.wsgi

                <Directory /var/www/FlaskApp/FlaskApp/>
                        WSGIProcessGroup main
                        WSGIApplicationGroup %{GLOBAL}
                        WSGIScriptReloading On
                        Order allow,deny
                        Allow from all
                </Directory>

                ErrorLog ${APACHE_LOG_DIR}/error.log
                LogLevel warn
                CustomLog ${APACHE_LOG_DIR}/access.log combined
				</VirtualHost>
				
			5. Enable Virtual host
					sudo a2ensite FlaskApp
			6. Create .wsgi file
			
				sudo vim /var/www/FlaskApp/FlaskApp/main.wsgi
				#!/usr/bin/python
				import sys
				import logging
				logging.basicConfig(stream=sys.stderr)
				sys.path.append('/var/www/FlaskApp/FlaskApp/')

				from main import app as application
				application.secret_key = 'itsasecret'
				
			7. Restart Apache service
				sudo service apache2 restart 

10. Configure the local timezone to UTC
	#List the current date time  settings 
			timedatectl
	#If it is not UTC run 
			sudo dpkg-reconfigure tzdata
	#select None of the above and then select UTC from there and sellect ok
	


Third-party Tools and Resources Used to Complete This Project

	1.Tools
		Amazon Lightsail -> To Host the Web Application
		MobaXterm SSH clinent -> For remort Access
		Notepad++ -> for editting
		Vagrant VM -> for testing
	2.Reference
		-> How To Set Up SSH Keys : https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys--2
		-> How To Install and Use PostgreSQL : https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-16-04
		-> How To Secure PostgreSQL : https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps
		-> SQLAlchemy : http://docs.sqlalchemy.org/en/latest/core/engines.html
		-> Managing PostgreSQL Databases : https://www.postgresql.org/docs/8.2/static/managing-databases.html
		-> Running a Flask application under the Apache WSGI module : https://www.jakowicz.com/flask-apache-wsgi/
		

# Linux Server Configuration

- IP address: `52.26.213.235`
- URL: [http://ec2-52-26-213-235.us-west-2.compute.amazonaws.com](http://ec2-52-26-213-235.us-west-2.compute.amazonaws.com)
- SSH port: `2200`
- to ban attackers I have used `fail2ban`
- to keep the system up to date I set up the `unattended-upgrades` package

### Software

	apt-get install apache2
	apt-get install apache2-doc
	apt-get install libapache2-mod-wsgi
	apt-get install postgresql
	apt-get install git
	apt-get install python-pip
	apt-get install python-psycopg2
	apt-get install fail2ban
	apt-get install unattended-upgrades

### Configuration

##### grant sudo privileges to `grader` user

	usermod -a -G sudo grader

##### SSH port

###### edit /etc/ssh/sshd_config

	# What ports, IPs and protocols we listen for
	Port 2200

##### Firewall (ufw)

###### enable ufw and only allow ports 80, 123 and 2200

	ufw default deny

	ufw allow 80
	ufw allow 123
	ufw allow 2200

	ufw enable

##### fail2ban

###### enable ufw logging for fail2ban

	ufw logging on

###### install fail2ban

	apt-get install fail2ban

##### enable automatic (security) updates

###### install unattended-upgrades

	apt-get install unattended-upgrades

###### edit /etc/apt/apt.conf.d/10periodic so that the upgrades are downloaded and installed every day

	APT::Periodic::Update-Package-Lists "1";
	APT::Periodic::Download-Upgradeable-Packages "1";
	APT::Periodic::AutocleanInterval "7";
	APT::Periodic::Unattended-Upgrade "1";

##### PostgreSQL

###### create db-user catalog
	sudo -u postgres createuser -P -d catalog

###### create database catalog with owner catalog
	sudo -u postgres createdb -O catalog catalog

##### Apache

###### create catalog.conf in /etc/apache2/sites-available

	<VirtualHost *:80>
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

###### create catalog.wsgi in /var/www/catalog:

	#!/usr/bin/python
	import sys
	import logging
	import os
	logging.basicConfig(stream=sys.stderr)
	sys.path.insert(0,"/var/www/catalog/")
	os.chdir("/var/www/catalog/catalog/")
	
	from catalog import app as application
	application.secret_key = 'the secret key'

###### deactivate default website, activate catalog

	a2dissite 000-default
	a2ensite catalog

##### catalog app

###### install catatlog dependencies

	pip install virtualenv 

	pip install Flask
	pip install SQLAlchemy
	pip install Flask-GoogleLogin

###### clone catalog repository to `/var/www/catalog/catalog`

###### set up the environment as described [here](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)

###### update SQLAlchemy connection string in `__init__.py` and `database_setup.py`

	postgresql+psycopg2://catalog:[catalog_user_password]@localhost:5432/catalog

###### update the authorized URLs in the google developers console

	http://ec2-52-26-213-235.us-west-2.compute.amazonaws.com/oauth2callback/

### Third-party resources

- [http://askubuntu.com/questions/168280/how-do-i-grant-sudo-privileges-to-an-existing-user](http://askubuntu.com/questions/168280/how-do-i-grant-sudo-privileges-to-an-existing-user)
- [http://linuxlookup.com/howto/change_default_ssh_port](http://linuxlookup.com/howto/change_default_ssh_port)
- [https://wiki.ubuntuusers.de/ufw](https://wiki.ubuntuusers.de/ufw)
- [https://help.ubuntu.com/community/UbuntuTime](https://help.ubuntu.com/community/UbuntuTime)
- [https://wiki.ubuntuusers.de/apache](https://wiki.ubuntuusers.de/apache)
- [https://wiki.ubuntuusers.de/Apache/mod_wsgi](https://wiki.ubuntuusers.de/Apache/mod_wsgi)
- [http://wiki.ubuntuusers.de/postgresql](http://wiki.ubuntuusers.de/postgresql)
- [http://flask.pocoo.org/docs/0.10/deploying/mod_wsgi/](http://flask.pocoo.org/docs/0.10/deploying/mod_wsgi/)
- [http://initd.org/psycopg/docs/install.html](http://initd.org/psycopg/docs/install.html)
- [http://docs.sqlalchemy.org/en/latest/dialects/postgresql.html#module-sqlalchemy.dialects.postgresql.psycopg2](http://docs.sqlalchemy.org/en/latest/dialects/postgresql.html#module-sqlalchemy.dialects.postgresql.psycopg2)
- [http://flask.pocoo.org/snippets/99/](http://flask.pocoo.org/snippets/99/)
- [https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
- [https://discussions.udacity.com/t/oauth-provider-callback-uris/20460](https://discussions.udacity.com/t/oauth-provider-callback-uris/20460)
- [http://weworkweplay.com/play/setting-up-your-own-linux-server-part-1-security/](http://weworkweplay.com/play/setting-up-your-own-linux-server-part-1-security/)
- [https://help.ubuntu.com/lts/serverguide/automatic-updates.html](https://help.ubuntu.com/lts/serverguide/automatic-updates.html)


	 
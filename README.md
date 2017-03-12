# Linux-server-configuration

- IP address: 184.72.127.153

- SSH port: 2200

- URL: http://184.72.127.153/

### Steps

1. Create an instance on https://lightsail.aws.amazon.com/

2. Update currently installed packages
  - sudo apt-get update
  - sudo apt-get upgrade

3. Change the SSH port from 22 to 2200
  - Run `sudo nano /etc/ssh/sshd_config`
  - Change the port from 22 to 2200

4. Configure the Uncomplicated Firewall (UFW) 
   - `sudo ufw allow 2200/tcp`
  - `sudo ufw allow 80/tcp`
  - `sudo ufw allow 123/udp`
  - `sudo ufw enable`

5. Create a new user account named grader
  - Run `$ sudo adduser grader` to create new user grader

6. Give grader the permission to sudo
  - Create new file in the sudoers directory with `sudo nano /etc/sudoers.d/grader`
  - Add the following text `grader ALL=(ALL:ALL) ALL`

7. Create an SSH key pair for grader using the ssh-keygen tool
  - `mkdir .ssh`
  - `touch .ssh/authorized_keys`
  - `nano .ssh/authorized_keys - Copy local ssh key to this file`
  - `chmod 700 .ssh`
  - `chmod 644 .ssh/authorized_keys`

8. Configure the local timezone to UTC
  - Run `sudo dpkg-reconfigure tzdata` and then choose UTC

9. Install and configure Apache to serve a Python mod_wsgi application
  - Run `sudo apt-get install apache2`
  - Run `sudo apt-get install libapache2-mod-wsgi python-dev`
  - Run `sudo a2enmod wsgi`
  - Run `sudo service apache2 start`


10 - Clone the Catalog app
  1. `$ cd /var/www`. Then: `$ sudo mkdir catalog`.
  2. Change owner for the *catalog* folder: `$ sudo chown -R grader:grader catalog`.
  3. Clone repository from Github: `$ git clone https://github.com/bassutandur/item_catalog catalog`.
  4. Make a *catalog.wsgi* file to serve the application over the *mod_wsgi*. That file should look like this:
```python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
```

11 - Install virtual environment, Flask and the project's dependencies

  1. Install pip, the tool for installing Python packages: `$ sudo apt-get install python-pip`.
  2. If *virtualenv* is not installed, use *pip* to install it using the following command: `$ sudo pip install virtualenv`.
  3. Install Flask: `$ pip install Flask`.
  4. Install all the other project's dependencies: `$ pip install bleach httplib2 request oauth2client sqlalchemy python-psycopg2`.

12 - Configure and enable a new virtual host

  1. Create a virtual host conifg file: `$ sudo nano /etc/apache2/sites-available/catalog.conf`.
  2. Paste in the following lines of code:
```
<VirtualHost *:80>
	ServerName 184.72.127.153
	ServerAlias 184.72.127.153
	ServerAdmin admin@184.72.127.153
	WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
	WSGIProcessGroup catalog
	WSGIScriptAlias / /var/www/catalog/catalog.wsgi
	<Directory /var/www/catalog/catalog/catalog/>
	Order allow,deny
	Allow from all
	</Directory>
	Alias /static /var/www/catalog/catalog/catalog/static
	<Directory /var/www/catalog/catalog/catalog/static/>
	Order allow,deny
	Allow from all
	</Directory>
	ErrorLog ${APACHE_LOG_DIR}/error.log
	LogLevel warn
	CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```


13. Install and configure PostgreSQL:
  - `sudo apt-get install postgresql postgresql-contrib`
  - `sudo su - postgres`
  - `CREATE USER catalog WITH PASSWORD 'password';`
  - `ALTER USER catalog CREATEDB;`
  - `ALTER USER catalog CREATEDB;`
  - `CREATE DATABASE catalog WITH OWNER catalog;;`
  - `\c catalog;`
  - `REVOKE ALL ON SCHEMA public FROM public;`
  - `GRANT ALL ON SCHEMA public TO catalog;`
- Inside the Flask application, the database connection is now performed with:
  - engine = create_engine('postgresql://catalog:password@localhost/catalog')
  - Setup the database with: `python /var/www/catalog/catalog/setup_database.py`

14. Install git
  - `sudo apt-get install git`


15. Clone and setup your Item Catalog project

16. Restart Apache
  - `sudo service apache2 restart`

17. Visit site at [http://184.72.127.153/]

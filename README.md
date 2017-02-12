# Linux Server Configuration
```
IP 52.32.235.31
URL: http://52.32.235.31
grader passwd: 'password'
```
[Bellagora](http://52.32.235.31)

## Project Requirements

1. Create a new user named grader
2. Give the grader the permission to sudo
3. Update all currently installed packages
4. Change the SSH port from 22 to 2200
5. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections
   for SSH (port 2200), HTTP (port 80), and NTP (port 123)
6. Configure the local timezone to UTC
7. Install and configure Apache to serve a Python mod_wsgi application
8. Install and configure PostgreSQL. Do not allow remote connections.
9. Create a new user named catalog that has limited permissions to your catalog
    application database
10. Install git, clone and setup your Catalog App project (from your GitHub
    repository from earlier in the Nanodegree program) so that it functions
    correctly when visiting your server’s IP address in a browser.

## Creating user 'grader':

$ sudo adduser grader

## Giving grader sudo permissions

$ sudo visudo

Inserted:
```
  'grader ALL=(ALL:ALL) ALL' in User privilege specification.
  ',timestamp_timeout=3,tty_tickets' after 'env_reset' to ensure prompts
  for passwords for grader usage of sudo.
```
## Update currently installed packages

$ sudo apt-get update
$ sudo apt-get upgrade

## Changing ssh port from 22 to 2200

$ sudo vim /etc/ssh/sshd_config

  Changed:
  ```
  Port: 20 to Port 2200
  ```
## Configuring the uncomplicated firewall

$ sudo ufw default deny incoming

$ sudo ufw default allow outgoing

$ sudo ufw allow 2200/tcp

$ sudo ufw allow www

$ sudo ufw allow ntp

$ sudo ufw enable

## Configuring local timezone

$ sudo dpkg-reconfigure tzdata

Selected:
  UTC

## Install and configure Apache and mod_wsgi

$ sudo apt-get install apache2

$ sudo apt-get install libapache2-mod-wsgi.

$ sudo vim /etc/apache2/sites-enabled/000-default.conf

  Added:
  ```
  "WSGIScriptAlias / /var/www/html/bellagora.wsgi"
  immediately before closing tag (wsgi file to be created later)
   ```
## Install and configure PostgreSQL, creating user 'catalog'

sudo apt-get install postgresql

sudo -i -u postgres

$ psql
  In Postgres:
  ```
  # CREATE USER catalog;
  # ALTER USER catalog WITH PASSWORD udacity2017;
  # CREATE DATABASE stock;
   ```
$ cat /etc/postgresql/9.3/main/pg_hba.conf

  Not accepting remote connections.

## Setting up Catalog App project

$ sudo apt-get install git

$ sudo apt-get install python-pip

$ pip install virtualenv

$ cd /var/www

$ virtualenv bellagora

$ cd bellagora

Repository "Bellagora" cloned into bellagora virtual environment. 
Virtual environment activated for the installation of required packages for application:

$ source bin/activate

$ sudo apt-get install python-dev

$ sudo apt-get install libpq-dev

$ sudo apt-get install python-psycopg2

$ pip install -r requirements.txt

$ deactivate

Database's name changed in files app.py, database_setup.py, and testdb.py.
While editing app.py, client_secrets.json and fb_client_secrets.json were
extended to their full paths.
```
  /var/www/bellagora/bellagora/client_secrets.json
  /var/www/bellagora/bellagora/fb_client_secrets.json
```
$ python database_setup.py

$ python testdb.py

$ sudo vim /etc/apache2/sites-available/Bellagora.conf

Added:
```
  <VirtualHost 'asterisk':80>
    ServerName 52.32.235.31
    ServerAdmin gibbirdsong@gmail.com
    WSGIScriptAlias / /var/www/bellagora/bellagora.wsgi
    <Directory /var/www/bellagora/bellagora/>
      Order allow,deny
      Allow from all
    </Directory>
    Alias /static /var/www/bellagora/bellagora/static
    <Directory /var/www/bellagora/bellagora/static/>
      Order allow,deny
      Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
  </VirtualHost>
```
Creating an wsgi file for catalog application:

$ sudo vim /var/www/bellagora/bellagora.wsgi

Added:
```
  #!/usr/bin/python
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0,"/var/www/bellagora/bellagora")

  from app import app as application
  application.secret_key = 'everythingallthetime'
```

$ sudo a2ensite Bellagora

$ sudo service apache2 restart

## Miscellaneous

To eliminate warning "unable to resolve host":

$ sudo vim etc/hosts

Added:
  ```
  127.0.0.1 ip-10-20-15-236
  ```

To force ssh key based authentication:

$ sudo vim /etc/ssh/sshd_config

Confirmed:
```
  PasswordAuthentication set to 'no'
```

To disable remote login of root:

$ sudo vim /etc/ssh/sshd_config

Changed:
```
  PermitRootLogin: 'withoutpassword' --> 'no'
 ```
Inserted:
```
  AllowUsers grader
```
## Sources:

Disabling remote logging of root:
```
https://www.a2hosting.com/kb/getting-started-guide/accessing-your-account/disabling-ssh-logins-for-root
```
Setting up a wsgi file:
```
http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/
```
Setting up PostgreSQL:
```
http://www.techrepublic.com/blog/diy-it-guy/diy-a-postgresql-database-server-setup-anyone-can-handle/
```
Working with Flask and Apache:
```
https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
```
Setting timestamp_timeout for sudoers:
```
http://askubuntu.com/questions/153933/no-password-prompt-at-sudo-command
```
And of course, the Udacity course material was heavily referenced.

```
https://classroom.udacity.com/nanodegrees/nd004/parts/00413454014/modules/357367901175461/lessons/4378692847/concepts/48114089370923#
```

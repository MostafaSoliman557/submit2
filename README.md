
# Linux Server Configuration
In this project we will take a baseline installation of a Linux server and prepare it to host our web applications. We will secure our server from a number of attack vectors, install and configure a database server, and deploy one of our existing web applications onto it.

* IP ADDRESS:3.122.204.14
* user : grader
* SSH port: 2200
* Complete URL to the hosted web application: http://3.122.204.14/




### Update all currently installed packages
```
$ sudo apt-get update
$ sudo apt-get upgrade
```
### Change the SSH port from 22 to 2200
```
$ sudo nano /etc/ssh/sshd_config
```
* Then change port 20 to 2200, save and exit.

* After that we need to reload SSH
```
$ sudo service ssh restart
```

### Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
* Block all the incoming requests

```
$ sudo ufw default deny incoming 
```
* Allow all the outgoing requests
```
$ sudo ufw default allow outgoing
```
* Allow incoming requests to specified ports
```
$ sudo ufw allow 2200/tcp
$ sudo ufw allow www
$ sudo ufw allow 123/udp 
```
* Activate the ufw firewall
```
$ sudo ufw enable
```
* Now, if we run  ```  $ sudo ufw status ```
```
To                         Action      From
--                         ------      ----
2200/tcp                   ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
123/udp                    ALLOW       Anywhere
2200/tcp (v6)              ALLOW       Anywhere (v6)
80/tcp (v6)                ALLOW       Anywhere (v6)
123/udp (v6)               ALLOW       Anywhere (v6)

```
### Disable root remote login
```
$ sudo nano /etc/ssh/sshd_config
```
* Change ```PermitRootLogin prohibit-password```  to  ```PermitRootLogin no```

### Create grader user
```
$ sudo adduser grader
```
### Give sudo access to grader user
```
$ sudo cp /etc/sudoers.d/90-cloud-init-users /etc/sudoers.d/grader

$ sudo nano /etc/sudoers.d/grader
```
* Replace ```ubuntu ALL=(ALL) NOPASSWD:ALL``` with ```grader ALL=(ALL) NOPASSWD:ALL```

### Create an SSH key pair for grader using the ssh-keygen tool

* On the local machine type this command
```
$ ssh-keygen
```
* It will ask for a password to add another layer of security

* On the server switch to the grader user and make sure you're in the home directory ```cd ~ ```
* Make ```.ssh``` directory which will store all your key related files
```
$ mkdir .ssh
```
* Create new file within ```.ssh``` directory called ```authorized_keys``` that will store all the public keys that this user is allowed to use for authentication
```
$ touch .ssh/authorized_keys
```

* Switch again to your local machine and copy the content of the public key file which has the extension ```.pub``` and paste it in ```authorized_keys``` file in your server

### Change the permissions of ```.ssh``` directory and ```authorized_keys``` file
```
$ chmod 700 .ssh
$ chmod 644 .ssh/authorized_keys
```

### Login to grader user
```
$ ssh grader@instanceIP -p SSH-port -i ~/.ssh/keys-file
```

### Configure the local timezone to UTC
```
$ sudo dpkg-reconfigure tzdata
```
* Choose ```none of the above from the first list```
* Choose ```UTC``` from the second list

### Install and configure Apache to serve a Python mod_wsgi application
```
$ sudo apt-get install apache2
$ sudo apt-get install libapache2-mod-wsgi python-dev
$ sudo service apache2 restart
```

### Install and configure PostgreSQL
* Install PostgreSQL
```
$ sudo apt-get install postgresql
```
* To make sure no remote connections are allowed we can check this file 
```
$ sudo nano /etc/postgresql/9.5/main/pg_hba.conf
```

* During the Postgres installation, an operating system user named postgres was created to correspond to the postgres PostgreSQL administrative user. We need to change to this user to perform administrative tasks
```
$ sudo su - postgres
```
* Get into postgreSQL shell 
```
$ psql
```
* Create a new database named ```catalog```
```
$ postgres=# CREATE DATABASE catalog;
```
* Create a new user name ```catalog```
```
$ postgres=# CREATE USER catalog;
```
* Set a password for the user ```catalog```
```
$ postgres=# ALTER ROLE catalog WITH PASSWORD '1234';
```
* Give our database user access rights to the database we created

```
$ postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
```
* To exit postgreSQL shell 
```
$ postgres=# \q
```
* To logout from ```postgres``` user
```
$ exit
```

### Install git
```
$ sudo apt-get install git
```

### Install packages needed for the project to work
* Install Flask
```
$ sudo apt-get install python-flask

```
* Install SQLalchemy
```
$ sudo apt-get install python-sqlalchemy
```

* Install requests module
```
$ sudo apt-get install python-requests
```
* Install PostgreSQL database adapter for the Python 
```
$ sudo apt-get install python-psycopg2
```
* Install http client library
```
$ sudo apt-get install python-httplib2
```
* Install oauth2 client library
```
$ sudo apt-get install python-oauth2client
```

### Deploy the Item Catalog project
* Move to the /var/www directory
```
$ cd /var/www
```

* Create a directory that will contain the application

```
$ sudo mkdir FlaskApp
```
* Make ubuntu user the owner of that folder in order to be able to clone the app inside it
```
$ sudo chown ubuntu /var/www/FlaskApp
```
* Move inside the newly created directory
```
$ cd FlaskApp
```
* Clone the Item Catalog app repository
```
$ git clone https://github.com/EMahmoudNabil/Item-Catalog-master.git
```
* Change the app name to ```FlaskApp```
```
$ sudo mv ./Item-Catalog-master ./FlaskApp
```
* Create and edit ```/etc/apache2/sites-available/FlaskApp.conf```
```
$ sudo nano /etc/apache2/sites-available/FlaskApp.conf
```

* Paste the following in it
```
<VirtualHost *:80>
        ServerName 3.122.204.14
        ServerAdmin admin@mywebsite.com
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

* Create and edit ```/var/www/FlaskApp/flaskapp.wsgi ```
```
$ sudo nano /var/www/FlaskApp/flaskapp.wsgi
```
* Paste the following in it
```
#! /usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/FlaskApp/FlaskApp/")

# home points to the home.py file
from server import app as application
application.secret_key = "somesecretsessionkey"

```

* Edit the line that read from the client secrets file 

```
CLIENT_ID = json.loads(open('/var/www/FlaskApp/FlaskApp/client_secrets.json', 'r').read())['web']['client_id']
```
* Restart Apache
```
$ sudo service apache2 restart
```
* Make ```.git``` directory not publicly accessible via a browser
```
$ sudo nano /etc/apache2/conf-enabled/security.conf
```
* Change this section
```
# Forbid access to version control directories
#
# If you use version control systems in your document root, you should
# probably deny access to their directories. For example, for subversion:
#
#<DirectoryMatch "/\.svn">
# Require all denied
#</DirectoryMatch>
```
* To this section
```
# Forbid access to version control directories
#
# If you use version control systems in your document root, you should
# probably deny access to their directories. For example, for subversion:
#
<DirectoryMatch "/\.git">
Require all denied
</DirectoryMatch>

```
* Restart Apache again
```
$ sudo service apache2 restart
```

### Resources
* [Amazon EC2 Linux Instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html)
* [Flask mod_wsgi (Apache)](http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/)
* [Apache Server Configuration Files](https://httpd.apache.org/docs/current/configuring.html)
* [Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
* [Set Up Apache Virtual Hosts on Ubuntu ](https://www.digitalocean.com/community/tutorials/how-to-set-up-apache-virtual-hosts-on-ubuntu-14-04-lts)
* [mod_wsgi documentation](https://modwsgi.readthedocs.io/en/develop/)
* [Automatic Security Updates](https://help.ubuntu.com/community/AutomaticSecurityUpdates#Using_the_.22unattended-upgrades.22_package)
* [Protect SSH with Fail2Ban](https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-14-04)
* [UFW with Fail2ban](https://askubuntu.com/questions/54771/potential-ufw-and-fail2ban-conflicts)
* [Fix locale issue](https://askubuntu.com/questions/162391/how-do-i-fix-my-locale-issue)
* [Ask Ubuntu](https://askubuntu.com/)
* [Stack Overflow](https://stackoverflow.com/)

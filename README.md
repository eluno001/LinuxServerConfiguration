# Linux Server Configuration project Udacity

## Server Details

### Access
IP adress: 35.177.41.0  
SSH port: 2200  
URL: I did not link this app to a DNS. It can still be accessed using the IP
adress. http://35.177.41.0/  

### Summary of software I installed
* Apache
* WSGI
* Flask
* PostgreSQL
* pip package
* oauth2client
* requests


### Configuration changes
After signing up and creating an ubuntu instance on Lightsail, we update the
packages:
```
sudo apt-get update
sudo apt-get upgrade
```

We change the ssh port 2200 by fist editing the file sshd_config:
```
sudo nano /etc/ssh/sshd_config
```
We changed the port number to 2200
We also allowed created a custom port in the networking tab on the Lightsail
plateform.

We configure the Uncomplicated Firewall (UFW) to only allow incoming
connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
```
sudo ufw allow 2200/tcp
sudo ufw allow www
sudo ufw allow 123/tcp
sudo ufw enable
```

We create a user named 'grader':
```
sudo adduser grader
```

We grant the grader sudo access by creating file in /etc/sudoers.d called
grader and we open the file for editing
```
sudo touch /etc/sudoers.d/grader
sudo nano /etc/sudoers.d/grader
```
We add the following line and save the file:
```
grader ALL=(ALL) NOPASSWD:ALL
```
We create on the local machine a key pair to allow authentication for the
grader user.
```
ssh-keygen
```
This generates two files: graderUdacityKey.pub and graderUdacityKey.
I need to place the public key (the .pub file content) inside the server.
```
sudo mkdir /home/grader/.ssh
sudo touch /home/grader/.ssh/authorized_keys
sudo nano /home/grader/.ssh/authorized_keys
```
Then we add the content of graderUdacityKey.pub and we save.  
We also need to set file permissions:
```
chmod 700 /home/grader/.ssh
chmod 644 /home/grader/.ssh/authorized_keys
```

We make sure to disable password authentication. Apparently on Lightsail,
it is disabled by default. We can check that in the sshd_config file. We also
disable remote login of root:
```
sudo nano /etc/ssh/sshd_config
```
We look for the following line and set it to no if it is not already.
```
# Change to no to disable tunnelled clear text passwords
PasswordAuthentication no
```
and 
```
# Authentication:
LoginGraceTime 120
PermitRootLogin no
StrictModes yes
```

We set the local timezone to utc
```
sudo dpkg-reconfigure tzdata
```
We scroll down to 'none of the above' and then choose UTC.

We install apache
```
sudo apt-get install apache2 
```
We check that the application has been properly installed by going to 
http://35.177.41.0/  

We install an application handler that Apache will be handing-off some  
requests to.
```
sudo apt-get install libapache2-mod-wsgi python-dev
```
We make sure that WSGI is enabled we use the following command:
```
sudo a2enmod wsgi 
```

We install pip
```
sudo apt-get install python-pip
```
After a failed attempt to install using pip. I understood that I should make 
a minor modification using:
```
export LC_ALL="en_US.UTF-8"
sudo dpkg-reconfigure locales
```
Then, we install virtualenv using pip. This will allow keep the application 
and its dependencies isolated from the main system.
```
sudo pip install virtualenv 
```
We install git
```
sudo apt-get install git
```
We clone the repository we for the catalog app and procede to few changes.
I chose to make those changes directly on the server.  
First we rename the project.py file to __init__.py  
Then, inside the __init__.py file we change  
```
engine = create_engine('sqlite:///items_catalog_with_users.db')
```
into
```
engine = create_engine('postgresql://catalog:catalog@localhost/catalog')
```
We change refer to the client_secrets.json file using an absolute path:
```
/var/www/CatalogApp/CatalogApp/client_secrets.json
```
We change:
```
app.run(host='0.0.0.0', port=5000)
```
into
```
app.run()
```

We install requests:
```
apt-get install python-requests

```

We create a temporary environment:
```
sudo virtualenv venv
```
We activate the environment using:
```
source venv/bin/activate
```
We install flask inside the environment
```
sudo pip install Flask
```
We can test if the installation was successful using
```
sudo python __init__.py 
```
The response was: http://127.0.0.1:5000/. So the installation was successful.  
We deactivate the environment:
```
deactivate
```
We Configure and Enable a New Virtual Host
```
sudo nano /etc/apache2/sites-available/CatalogApp.conf
```
We add configuration to that file.
```
<VirtualHost *:80>
                ServerName 35.177.41.0
                ServerAdmin myemail@dsdhjf.com
                WSGIScriptAlias / /var/www/CatalogApp/CatalogApp.wsgi
                <Directory /var/www/CatalogApp/CatalogApp/>
                        Order allow,deny
                        Allow from all
                </Directory>
                Alias /static /var/www/CatalogApp/CatalogApp/static
                <Directory /var/www/CatalogApp/CatalogApp/static/>
                        Order allow,deny
                        Allow from all
                </Directory>
                ErrorLog ${APACHE_LOG_DIR}/error.log
                LogLevel warn
                CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
We save and close the file.
We enable the virtual host with the following command:
```
sudo a2ensite CatalogApp
```
We create the CatalogApp.wsgi file that will serve the Flask app.  
```
cd /var/www/CatalogApp
sudo nano CatalogApp.wsgi 
```
We paste the following:
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/CatalogApp/")

from CatalogApp import app as application
application.secret_key = 'Add your secret key'
```
We install PostgreSQL
```
sudo apt-get install postgresql
```
We make sure that postgresql is not accessible remotely.
```
sudo nano /etc/postgresql/9.5/main/pg_hba.conf
```
We create a new user
```
sudo su - postgres 
psql 
CREATE DATABASE catalog;  
CREATE USER catalog WITH PASSWORD 'catalog';  
```
We make sure that the user has limited privileges
```
\du
```
We configure google signin to accept requests from: http://35.177.41.0 

We also delete .git directory from /var/www/CatalogApp/CatalogApp for it
not to be publicly available.

## References:  
https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps#step-one—-install-and-enable-mod_wsgi  
https://www.youtube.com/watch?v=-LwI4HMR_Eg&t=715s  
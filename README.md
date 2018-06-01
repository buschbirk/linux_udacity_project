# Linux Server Project - Udacity

This project uses a Google Cloud virtual machine running Ubuntu to host
the website for a previous Udacity project using python Flask, Postgresql and
Apache.

## Getting Started

I created a new virtual machine running Ubuntu in the Google Cloud platform.
I used the Google Cloud platform to 'SSH' into the VM initially

## Update packages and add user 'grader'
Type in the following commands:
```
# Ensure that timezone is UTC
time
# update packages and install git and Python pip
sudo apt-get update
sudo apt-get upgrade
sudo apt install git-all
sudo apt install python3-pip
sudo adduser grader
# Give super-user permissions to user 'grader'
sudo adduser grader sudo
```

## Changing SSH port to 2200 and changing firewall settings
Type ```sudo nano /etc/ssh/sshd_config``` and change 'port 22' to 'port 2200'
Set up firewall:
```
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2200 #ssh traffic
sudo ufw allow http #web traffic on port 80
sudo ufw allow 123 # NTP
sudo ufw enable
service sshd restart #restart SSH service
```
In your Google Cloud console, locate the 'Network interface details'
and change the default SSH port 22 to 2200

## Installing Postgres and configuring users
Type in the following command to install Postgres:
```
sudo apt-get install postgresql
```
Use ```nano /etc/postgresql/10/main/pg_hba.conf``` to check that
there are no external connections allowed for your SQL database

Configure postgres users:
```
sudo -u postgres psql postgres
\password postgres
# create new postgres user and set it's permissions to enable creation of tables
CREATE ROLE catalog WITH LOGIN PASSWORD 'catalogPass';
ALTER USER catalog CREATEDB;
# Exit out of postgresql
\q
# restart postgresql
sudo service postgresql restart
```

## Cloning the project and populating database
Type in the following commands:
```
sudo mkdir /var/www/FLASKAPP
cd /var/www/FLASKAPP
git clone https://github.com/buschbirk/udacity_catalogue_project.git item_cat_app
# Install and activate a Python virtual environment
python3 -m pip install --user virtualenv
python3 -m virtualenv venv
source venv/bin/activate
# install Python packages
sudo pip3 install -r  requirements.txt
# Set up and populate the Postgres database table used by our Flask app
sudo python3 create_database.py
sudo python3 populate_item_catalogue.py
```

## Changes made in the item_cat_app directory
* Renamed "views.py" to ``__init__.py``
* Added the current working directory to sys.path in __init__.py
* Added new file item_cat_app.wsgi with the following content:
```
#!/usr/bin/python3
import sys
sys.path.insert(0,"/var/www/FLASKAPP/")
from item_cat_app import app as application
```

## Install and configue apache
Type in the following commands:
```
sudo apt-get install apache2
sudo apt-get install libapache2-mod-wsgi-py3
# check that apache is running by visiting your site's external IP address
```
Create new file /etc/apache2/sites-available/itemapp.conf and use the following configuration:
```
<VirtualHost *:80>
    ServerAdmin lasse@alsbirk.com
    ServerName 35.196.184.13

    WSGIDaemonProcess item_cat_app user=www-data group=www-data threads=5
    WSGIProcessGroup item_cat_app
    WSGIScriptAlias / /var/www/FLASKAPP/item_cat_app/item_cat_app.wsgi
    Alias "/static/" "/var/www/FLASKAPP/item_cat_app/static/"
    <Directory "/var/www/FLASKAPP/item_cat_app/static/">
        Order allow,deny
        Allow from all
    </Directory>

</VirtualHost>
```
Launch the site
```
sudo a2ensite itemapp.conf
systemctl reload apache2
```

## Create RSA key for external ssh access
Type in the following commands:
```
# imitate user 'grader'
sudo -i -u grader
# create keys
ssh-keygen
# Follow the instructions to save the key to the default path
```
Copy the public key located in 'home/grader/.ssh/id_rsa.pub' into the SSH keys list in the Metadata for your Google Cloud instance.
This will automatically add the key to the user's '.ssh/authorized_keys' file.

## Site details:
* Ip address: 35.196.184.13
* SSH port: 2200
* Website address: http://35.196.184.13/

## Built With
* [Google Cloud Console](https://cloud.google.com/)

## Author
* Lasse Alsbirk

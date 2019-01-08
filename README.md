# Linux Server Configuration

This page aims to explain the steps for the configuration of deploying and securing a Python web application on a Linux server using Amazon Lightsail. This project was done as a part of the Full Stack Web Developer Nanodegree on [Udacity](https://www.udacity.com/course/full-stack-web-developer-nanodegree--nd004)

The web application is deployed on this URL: http://ec2-18-194-41-12.eu-central-1.compute.amazonaws.com/

## Specifications of this project

1. Creating a Linux Server Instance on [Amazon Lightsail](https://lightsail.aws.amazon.com/)
2. Setting up Linux Distribution [Ubuntu](https://www.ubuntu.com/download/desktop) 16.04 LTS.
3. Using [Git](https://git-scm.com/) for cloning the project on the server
4. Using [PostgreSQL](https://www.postgresql.org/) as a database server
5. Deploying [Item Catalog](https://github.com/dalia-maher/Item-Catalog) Python Web Application which was created earlier in this Nanodegree program.

### Step 1: Create a Linux Server Instance on Amazon Lightsail

1. Crate an account on [Amazon Lightsail](https://lightsail.aws.amazon.com/).
2. Log in to Lightsail. If you don't already have an Amazon Web Services account, you'll be prompted to create one.
3. Once you're logged in, Lightsail will give you a friendly message with a robot on it, prompting you to create an instance. A Lightsail instance is a Linux server running on a virtual machine inside an Amazon datacenter.
4. Choose an instance image: "Unix/Linux" --> "OS only" --> "Ubuntu 16.04 LTS".
5. Choose an instance plan. The lowest tier of instance is just fine with this project.
6. Give your instance a hostname and wait for it to start up.

### Step 2: SSH into your server
1. From the `Account` menu at the navigation bar of Amazon Lightsail, choose the `SSH keys` tab and download the Default Private Key.
2. After you download the private key with a name similar to `LightsailDefaultKey-eu-central-1.pem`, open your terminal and change to the same directory which contains the key.
3. Run this command to change the file's permissions: `chmod 600 LightsailDefaultKey-eu-central-1.pem`.
4. Connect to the instance through the terminal by running this command: 
`ssh -i LightsailDefaultKey-eu-central-1.pem ubuntu@18.194.41.12` in which "18.194.41.12" is the Public IP of the instance and "ubuntu" is the username.

### Step 3: Update and upgrade all currently installed packages
1. Update the installed packages by running this command: `sudo apt-get update`.
2. Upgrade the installed packages by running this command: `sudo apt-get upgrade`.

### Step 4: Change the SSH port from 22 to 2200
1. Edit the `sshd_config` by running this command: `sudo nano /etc/ssh/sshd_config`
2. Change this line `Port 22` to be `Port 2200`.
3. Click on Ctrl + X to exit the editor then press `Y` to save the file.
4. Restart the SSH service by running this command: `sudo service ssh restart`.

### Step 5: Configure the Uncomplicated Firewall (UFW)
1. Run the following commands to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
  ```
  sudo ufw default deny incoming
  sudo ufw default allow outgoing
  sudo ufw allow 2200/tcp
  sudo ufw allow 80/tcp
  sudo ufw allow 123/udp
  ```
2. Enable the firewall by running this: `sudo ufw enable`.
3. Check the status of the UFW and make sure the ports are correct by running: `sudo ufw status`.
4. From the Amazon Ligthsail instance page, click on the `Networking` tab then, configure the ports to be the same like above.
5. Make sure you can connect to the server by logging in with the following command: 
`ssh -i LightsailDefaultKey-eu-central-1.pem -p 2200 ubuntu@18.194.41.12`.

### Step 6: Create a new user account named "grader"
1. You can create a user using this command: `sudo adduser grader` and then, enter the password desired for this user (twice).
2. Fill the information you want for this user.

### Step 7: Give "grader" the permission to sudo
1. Create a new directory in sudoer directory with `sudo nano /etc/sudoers.d/grader`.
2. Add `grader ALL=(ALL:ALL) ALL` in the file editor.
3. Verify that the grader has sudo permissions by running `su - grader` and entering the password then running `sudo -l`

### Step 8: Create an SSH key pair for "grader" using the "ssh-keygen" tool
1. Logout as the "grader" user to return to the local directory then run: `ssh-keygen`.
2. Give a name for the file. In my case, I named it `udacity_key`.
3. Enter a passphrase for that key (twice).
4. A file with the name `udacity_key.pub` is generated. Run `cat udacity_key.pub` and copy its contents from the terminal.
5. Login to the server then, create a new directory: `mkdir .ssh`.
6. Run the command: `sudo nano ~/.ssh/authorized_keys` then, paste the contents you've copied into the file and save it.
7. Set the permissions of the directory and the file by running: `chmod 700 .ssh` and `chmod 644 .ssh/authorized_keys`.
8. Open the `sshd_config` file: `sudo nano /etc/ssh/sshd_config` and make sure the `PasswordAuthentication` is set to `no` and disable the root login by setting `PermitRootLogin` to `no`.
9. Restart the SSH service: `sudo service ssh restart`.
10. Logout and try to login using ssh: `ssh -i udacity_key -p 2200 grader@18.194.41.12`.

### Step 9: Configure the local timezone to UTC
1. Configure the time zone: `sudo dpkg-reconfigure tzdata`.
2. Choose UTC as the time zone then, test it by running: `date`.

### Step 10: Install and configure Apache to serve a Python mod_wsgi application
1. Install Apache: `sudo apt-get install apache2`.
2. Confirm Apache is working by visiting http://18.194.41.12 in your browser. You should see the Apache Ubuntu Default Page.
3. Install mod_wsgi: `sudo apt-get install libapache2-mod-wsgi python-dev`.
4. Confirm mod_wsgi is working by running: `sudo a2enmod wsgi`

### Step 11: Install and configure PostgreSQL
1. Install PostgreSQL: `sudo apt-get install postgresql`.
2. Open the file `/etc/postgresql/9.5/main/pg_hba.conf file` and make sure it doesn't allow remote connections.
```
# Database administrative login by Unix domain socket
local   all             postgres                                peer

# TYPE  DATABASE        USER            ADDRESS                 METHOD
local   all             all                                     peer
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5
```
3. Switch to "postgres" user by running: `sudo su - postgres`.
4. Connect to PostgreSQL interactive terminal by writing: `psql`.
5. Create the "catalog" user and password: postgres=# `CREATE USER catalog WITH PASSWORD 'catalog';`.
6. Give the user the permission to create a database: postgres=# `ALTER USER catalog CREATEDB;`
7. Make sure of the existing roles by running: `\du`.
8. Create the database: postgres=# `CREATE DATABASE catalog WITH OWNER catalog;`
9. Make sure of the existing databases by running: `\l`.
10. Exit "psql" by running: `\q` then, `logout` to return to the grader's terminal.

### Step 12: Install "git"
1. Install git by running this command: `sudo apt-get install git`.

### Step 13: Clone and setup "Item Catalog" project from the Github repository
1. Make directory: `sudo mkdir /var/www/itemcatalog/` and change to it: `cd /var/www/itemcatalog/`.
2. Clone the Item Catalog project: `sudo git clone https://github.com/dalia-maher/Item-Catalog.git`.
3. Change the owner of the cloned directory: `sudo chown -R grader:grader itemcatalog/`.
4. Make sure that .git directory is not publicly accessible via a browser by running: `sudo nano /var/www/itemcatalog/.htaccess` and write in it: `RedirectMatch 404 /\.git` the, save & exit.
5. Change directory to the project's directory: `cd itemcatalog/`.
6. Rename `application.py` to `__init__.py` by running: `mv application.py __init__.py`.
7. Open the file: `sudo nano __init__.py` and replace the line at the end of the file `app.run(host='0.0.0.0', port=5000)` with `app.run()`.
8. Replace the lines of creating engine using Sqlite in `__init__.py` and `database_setup.py`
```
engine = create_engine('sqlite:///itemcatalog.db',
                       connect_args={'check_same_thread': False},
                       poolclass=StaticPool)
```
with PostgreSQL: 
```
engine = create_engine('postgresql://catalog:catalog@localhost/catalog')
```
9. Add the absolute file path for the client_secrets.json file in the __init__.py file. Change it from 'client_secrets.json' to '/var/www/itemcatalog/itemcatalog/client_secrets.json'.
10. Configure Google OAuth in the Google API Console by adding the server's domain and IP to the Web Client which was created for the project:
- From the "OAuth consent screen" tab, add ec2-18-194-41-12.eu-central-1.compute.amazonaws.com to the "Authorized domains".
- From the "Credentials" tab, open the OAuth client and put the following URIs to the "Authorized JavaScript origins":
```
http://18.194.41.12
http://ec2-18-194-41-12.eu-central-1.compute.amazonaws.com
```
- Also, put the following URIs to the "Authorized redirect URIs" then, save the changes:
```
http://ec2-18-194-41-12.eu-central-1.compute.amazonaws.com/oauth2callback
http://ec2-18-194-41-12.eu-central-1.compute.amazonaws.com/login
http://ec2-18-194-41-12.eu-central-1.compute.amazonaws.com/gconnect
```
11. Click on "Download JSON" at the top of the page to download the "client_secrets" file.
12. Open the "client_secrets" file downloaded then, copy its contents.
13. Return to the terminal and open the "client_secrets" file there: `sudo nano client_secrets.json`
14. Paste the contents to this file then, save & exit.

### Step 14: Setting up the project so that it functions correctly when visiting the server's IP address
1. Install pip: `sudo apt-get install python-pip`.
2. Install Virtual Environment: `sudo apt-get install python-virtualenv`.
3. Go to the project's directory: `cd /var/www/itemcatalog/itemcatalog/`.
4. Create virtual environment: `virtualenv venv`.
5. Activate the virtual environment by running: `. venv/bin/activate`.
6. Install the following packages as dependencies:
```
pip install flask
pip install sqlalchemy
pip install --upgrade oauth2client
pip install httplib2
pip install requests
pip install psycopg2
sudo apt-get install libpq-dev
pip install psycopg2-binary
```
7. Make sure that the application is error free and that it's runnung when you execute the following command: `python __init__.py`.
8. Deactivate the virtual environment by running `deactivate`.
9. Set up a virtual host by creating a file: `sudo nano /etc/apache2/sites-available/itemcatalog.conf` and add the following lines to it:
```
<VirtualHost *:80>
    ServerName 18.194.41.12
    ServerAlias ec2-18-194-41-12.eu-central-1.compute.amazonaws.com
    WSGIScriptAlias / /var/www/itemcatalog/itemcatalog.wsgi
    <Directory /var/www/itemcatalog/itemcatalog/>
        Order allow,deny
        Allow from all
    </Directory>
    Alias /static /var/www/itemcatalog/itemcatalog/static
    <Directory /var/www/itemcatalog/itemcatalog/static/>
        Order allow,deny
        Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
10. Change directory: `cd /etc/apache2/sites-available/` and run: `sudo a2ensite itemcatalog` to enable the virtual host.
11. Activate the new configuration by running: `service apache2 reload`.
12. Create the itemcatalog.wsgi file by running: `sudo nano /var/www/itemcatalog/itemcatalog.wsgi` and put the following lines to it then, save & exit:
```
activate_this = '/var/www/itemcatalog/itemcatalog/venv/bin/activate_this.py'
execfile(activate_this, dict(__file__=activate_this))

#!/usr/bin/python
import sys
import logging

logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/itemcatalog/")

from itemcatalog import app as application
application.secret_key = 'super_secret_key'
```
13. Restart Apache service: `sudo service apache2 restart`.
14. Disable the default Apache site: `sudo a2dissite 000-default.conf`.
15. Change directory: `cd /var/www` then, change the ownership of the project directory to the www-data user: `sudo chown -R www-data:www-data itemcatalog/`.
16. Restart the Apache service again: `sudo service apache2 restart`.
17. Finally, open the URL of the project http://ec2-18-194-41-12.eu-central-1.compute.amazonaws.com/ in a browser and make sure that the web application is working fine.

### Third-Party & Helpful Resources

- Udacity's course for Configuring Linux Servers: (Configuring Linux Web Servers)[https://www.udacity.com/course/configuring-linux-web-servers--ud299]
- (SSH: How to access a remote server and edit files)[https://www.youtube.com/watch?v=HcwK8IWc-a8]
- Useful command for checking server errors in the logs: `sudo tail /var/log/apache2/error.log`
- (Deploying on Linux Server using mod_wsgi and Virtual Environment)[http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/]
- (Deploying Flask Application - Quick Start)[http://flask.pocoo.org/docs/1.0/quickstart/]
- (How to deploy Flask Web Application on Ubuntu VPS)[https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps]
- (Configuring SQLAlchemy with Python using PostgreSQL)[https://docs.sqlalchemy.org/en/latest/core/engines.html]
- (Understanding Firewall and port mappings in Amazon Lightsail)[https://lightsail.aws.amazon.com/ls/docs/en/articles/understanding-firewall-and-port-mappings-in-amazon-lightsail]
- (How to secure PostgreSQL on Ubuntu VPS)[https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps]
- (Configuring UTC time)[https://help.ubuntu.com/community/UbuntuTime]
- (Making .git Directory inaccessible)[https://stackoverflow.com/questions/6142437/make-git-directory-web-inaccessible]
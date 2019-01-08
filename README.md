# Linux Server Configuration

This page aims to explain the steps for the configuration of deploying and securing a Python web application on a Linux server using Amazon Lightsail. This project was done as a part of the Full Stack Web Developer Nanodegree on [Udacity](https://www.udacity.com/course/full-stack-web-developer-nanodegree--nd004)

The web application is deployed on this URL: http://ec2-18-194-41-12.eu-central-1.compute.amazonaws.com/

## Specifications of this project

1. Creating a Linux Server Instance on [Amazon Lightsail](https://lightsail.aws.amazon.com/)
2. Setting up Linux Distribution [Ubuntu](https://www.ubuntu.com/download/desktop) 16.04 LTS.
3. Using [git](https://git-scm.com/) for cloning the project on the server
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
5. Make sure you can connect to your server by logging in with the following command: 
`ssh -i LightsailDefaultKey-eu-central-1.pem -p 2200 ubuntu@18.194.41.12`.

### Step 6: Create a new user account named "grader"
1. You can create a user using this command: `sudo adduser grader` and then, enter the password desired for this user (twice).
2. Fill the information you want for this user.

### Step 7: Give "grader" the permission to sudo
1. Create a new directory in sudoer directory with `sudo nano /etc/sudoers.d/grader`.
2. Add `grader ALL=(ALL:ALL) ALL` in the file editor.
3. Verify that the grader has sudo permissions by running `su - grader` and entering the password then running `sudo -l`

### Step 8: Create an SSH key pair for "grader" using the "ssh-keygen" tool
1. Logout as the "grader" user to return to your local directory then run: `ssh-keygen`.
2. Give a name for the file. In my case I named it `udacity_key`.
3. Enter a passphrase for that key (twice).
4. A file with the name `udacity_key.pub` is generated. Run `cat udacity_key.pub` and copy its contents from the terminal.
5. Login to the server then, create a new directory: `mkdir .ssh`.
6. Run the command: `sudo nano ~/.ssh/authorized_keys` then, paste the contents you've copied into the file and save it.
7. Set the permissions of the directory and the file by running: `chmod 700 .ssh` and `chmod 644 .ssh/authorized_keys`.
8. Open the `sshd_config` file: `sudo nano /etc/ssh/sshd_config` and make sure the `PasswordAuthentication` is set to `no`.
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
5. Create the "catalog" user and password: `postgres=# CREATE USER catalog WITH PASSWORD 'catalog';`.
6. Give the user the permission to create a database: `postgres=# ALTER USER catalog CREATEDB;`
7. Make sure of the existing roles by running: `\du`.
8. Create the database: `postgres=# CREATE DATABASE catalog WITH OWNER catalog;`
9. Make sure of the existing databases by running: `\l`.
10. Exit "psql" by running: `\q` then, `logout` to return to the grader's terminal.

### Step 12: Install "git"
1. Install git by running this command: `sudo apt-get install git`.

### Step 13: Clone and setup "Item Catalog" project from the Github repository
1. Make directory: `sudo mkdir /var/www/itemcatalog/` and change to it `cd /var/www/itemcatalog/`.
2. Clone the Item Catalog project: `sudo git clone https://github.com/dalia-maher/Item-Catalog.git`.
3. Change the owner of the cloned directory: `sudo chown -R grader:grader itemcatalog/`.
4. Change directory to the project's directory: `cd itemcatalog/`.
5. Rename `application.py` to `__init__.py` by running: `mv application.py __init__.py`.
6. Replace the line at the end of the file `app.run(host='0.0.0.0', port=5000)` with `app.run()`.
7. Replace the lines of creating engine using Sqlite in `__init__.py` and `database_setup.py`
```
engine = create_engine('sqlite:///itemcatalog.db',
                       connect_args={'check_same_thread': False},
                       poolclass=StaticPool)
```
with PostgreSQL: 
```
engine = create_engine('postgresql://catalog:catalog@localhost/catalog')
```
8. Configure Google OAuth in the Google API Console by adding the server's domain and IP to the Web Client which was created for the project:
- From the "OAuth consent screen" tab, add `ec2-18-194-41-12.eu-central-1.compute.amazonaws.com` to the "Authorized domains".
- From the "Credentials" tab, open the OAuth client and put the following URIs to the "Authorized JavaScript origins":
`http://18.194.41.12`
`http://ec2-18-194-41-12.eu-central-1.compute.amazonaws.com`
- Also, put the following URIs to the "Authorized redirect URIs" then, save the changes:
`http://ec2-18-194-41-12.eu-central-1.compute.amazonaws.com/oauth2callback`
`http://ec2-18-194-41-12.eu-central-1.compute.amazonaws.com/login`
`http://ec2-18-194-41-12.eu-central-1.compute.amazonaws.com/gconnect`
9. Click on "Download JSON" at the top of the page to download the 'client_secrets' file.
10. Open the 'client_secrets' file downloaded then, copy its contents.
11. Return to the terminal and open the 'client_secrets' file there: `sudo nano client_secrets.json`
12. Paste the contents to this file then, save & exit.
13. Add the absolute file path for the client_secrets.json file in the __init__.py file. Change it from 'client_secrets.json' to '/var/www/itemcatalog/itemcatalog/client_secrets.json'.

### Step 14: Setting up the project so that it functions correctly when visiting the server�s IP address


### Helpful Resources

- http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/
- 
# Linux Server Configuration

This page aims to explain the steps for the configuration of deploying and securing a Python web application on a Linux server using Amazon Lightsail. This project was done as a part of the Full Stack Web Developer Nanodegree on [Udacity](https://www.udacity.com/course/full-stack-web-developer-nanodegree--nd004)

The web application is deployed on this URL: http://ec2-18-194-41-12.eu-central-1.compute.amazonaws.com/

## Specifications of this project

1. Creating a Linux Server Instance on [Amazon Lightsail](https://lightsail.aws.amazon.com/)
2. Setting up Linux Distribution [Ubuntu](https://www.ubuntu.com/download/desktop) 16.04 LTS.
3. Using [git](https://git-scm.com/) for cloning the project on the server
4. Using [PostgreSQL](https://www.postgresql.org/) as a database server
5. Deploying [Item Catalog](https://github.com/dalia-maher/Item-Catalog) Python Web Application

### Step 1: Create a Linux Server Instance on Amazon Lightsail

1. Crate an account on [Amazon Lightsail](https://lightsail.aws.amazon.com/).
2. Log in to Lightsail. If you don't already have an Amazon Web Services account, you'll be prompted to create one.
3. Once you're logged in, Lightsail will give you a friendly message with a robot on it, prompting you to create an instance. A Lightsail instance is a Linux server running on a virtual machine inside an Amazon datacenter.
4. Choose an instance image: Unix/Linux --> OS only --> Ubuntu 16.04 LTS.
5. Choose an instance plan. The lowest tier of instance is just fine with this project.
6. Give your instance a hostname and wait for it to start up.

### Step 2: SSH into your server
1. From the `Account` menu at the navigation bar of Amazon Lightsail, choose the `SSH keys` tab and download the Default Private Key.
2. After you download the private key with a name similar to `LightsailDefaultKey-eu-central-1.pem`, open your terminal and change to the same directory which contains the key.
3. Run this command to change the file's permissions: `chmod 600 LightsailDefaultKey-eu-central-1.pem`.
4. Connect to the instance through by running this command: `ssh -i LightsailDefaultKey-eu-central-1.pem ubuntu@18.194.41.12` in which 18.194.41.12 is the public IP of the instance and ubuntu is the username.

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
3. Check the status and make sure the ports are correct by running: `sudo ufw status`.
4. From the Amazon Ligthsail instance page, click on the `Networking` tab then, configure the ports to be the same like above.
5. Make sure you can connect to your server by logging in with the following command: `ssh -i LightsailDefaultKey-eu-central-1.pem -p 2200 ubuntu@18.194.41.12`.

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
10. Logout and try to login using ssh: `ssh -i udacity_key -p 2200 grader@18.194.41.12`

### Helpful Resources

- http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/
- 
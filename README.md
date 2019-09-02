# linuxServerConfiguraton

## Project Overview
You will take a baseline installation of a Linux server and prepare it to host your web applications. You will secure your server from a number of attack vectors, install and configure a database server, and deploy one of your existing web applications onto it.

## Why this Project?
A deep understanding of exactly what your web applications are doing, how they are hosted, and the interactions between multiple systems are what define you as a Full Stack Web Developer. In this project, you’ll be responsible for turning a brand-new, bare bones, Linux server into the secure and efficient web application host your applications need.

## What will I Learn?
You will learn how to access, secure, and perform the initial configuration of a bare-bones Linux server. You will then learn how to install and configure a web and database server and actually host a web application.

## How will I complete this project?
This project is linked to the [Configuring Linux Web Servers course](https://classroom.udacity.com/courses/ud299), which teaches you to secure and set up a Linux server. By the end of this project, you will have one of your web applications running live on a secure web server.

To complete this project, you'll need a Linux server instance. We recommend using [Amazon Lightsail](https://signin.aws.amazon.com/signin?redirect_uri=https%3A%2F%2Flightsail.aws.amazon.com%2Fls%2Fwebapp%3Fstate%3DhashArgs%2523%26isauthcode%3Dtrue&client_id=arn%3Aaws%3Aiam%3A%3A015428540659%3Auser%2Fparksidewebapp&forceMobileApp=0) for this. If you don't already have an Amazon Web Services account, you'll need to set one up. Once you've done that, here are the steps to complete this project.

### Get your server.
1. Start a new Ubuntu Linux server instance on Amazon Lightsail. There are full details on setting up your Lightsail instance on the next page.
2. Follow the instructions provided to SSH into your server.
### Secure your server.
3. Update all currently installed packages.
4. Change the SSH port from 22 to 2200. Make sure to configure the Lightsail firewall to allow it.
5. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
### Give grader access.
In order for your project to be reviewed, the grader needs to be able to log in to your server.
6. Create a new user account named grader.
7. Give grader the permission to sudo.
8. Create an SSH key pair for grader using the ssh-keygen tool.
### Prepare to deploy your project.
9. Configure the local timezone to UTC.
10. Install and configure Apache to serve a Python mod_wsgi application.
 * If you built your project with Python 3, you will need to install the Python 3 mod_wsgi package on your server:
 * ```$ sudo apt-get install libapache2-mod-wsgi-py3```.
11. Install and configure PostgreSQL:
 * Do not allow remote connections
 * Create a new database user named catalog that has limited permissions to your catalog application database.
12. Install git.i
### Deploy the Item Catalog project.
13. Clone and setup your Item Catalog project from the Github repository you created earlier in this Nanodegree program.
14. Set it up in your server so that it functions correctly when visiting your server’s IP address in a browser. Make sure that your .git directory is not publicly accessible via a browser!

## Log in details for grader
* IP: 34.220.230.36
* Port: 2200
* URL: [ec2-34-220-230-36.us-west-2.compute.amazonaws.com](ec2-34-220-230-36.us-west-2.compute.amazonaws.com)

## Requirements
Below is a set of requirements with steps on how they were achieved
1. Github repository with [README.md](README.md).
2. Your README.md file should include all of the following:
    * i. The IP address and SSH port so your server can be accessed by the reviewer.
        * IP: 34.220.230.36 Port: 2200
    * ii. The complete URL to your hosted web application.    
        * URL: [ec2-34-220-230-36.us-west-2.compute.amazonaws.com/](ec2-34-220-230-36.us-west-2.compute.amazonaws.com/)
    * iii. A summary of software you installed and configuration changes made.
        * Linux Machine Software
            * Update: `sudo apt-get update`
            * Upgrade: `sudo apt-get upgrade`
            * Finger: `sudo apt-get install finger`
            * NTP: `sudo apt-get install ntp`
        * Grader User Setup
            * Create user: `sudo adduser grader`
            * Grant user permission to execute sudo command by running: `sudo vim /etc/sudoers.d/grader`
            * Add the following line in the file `grader ALL=(ALL) NOPASSWD:ALL`
        * Configure key-based authentication
            * Create ssh-key on your personal machine by running: `ssh-keygen -f ~/.ssh/someFilename`
            * Switch to the grader user by running: `su grader`
            * Copy the contents of the public key on your machine to the remote lightsail ubuntu machine `cat ~/.ssh/someFilename.pub`
            * Paste it in under the user grader's authorized_keys file: `vim ~/.ssh/authorized_keys`
            * Change the file permissions to the following: `sudo chmod 700 ~/.ssh` & `sudo chmod 644 ~/.ssh/authorized_keys`
            * Verify it is working by remote logging in to grader using the private key `ssh -i ~/.ssh/<your_private_key> grader@34.220.230.36`
       * SSH Configurations
            * Edit the following file `sudo vim /etc/ssh/sshd_config`
            * Disable password based authentication by giving the value of no to the Password Authentication variable ex: `PasswordAuthentication no`
            * Change SSH service port from 22 to 2200 by modifying the value of the Port variable ex: `Port 2200`
            * Disable SSH root log in by modifying the value of the value of the PermitRootLogin varibale ex: `PermitRootLogin no`
            * Restart the SSH service: `sudo service ssh restart`
       * Firewall Setup
            * `sudo ufw allow 2200/tcp`
            * `sudo ufw allow www`
            * `sudo ufw allow ntp`
            * `sudo ufw allow 'Apache Full'`
            * `sudo ufw allow https`
            * Make sure to enable same ports on EC2 lightsail portal under your instance > Netwroking > Firewall
            * `sudo ufw enable`
       * Apache Install & Other Server Requirements
            * `sudo apt-get install apache2`
            * Install mod-wsgi `sudo apt-get install libapache2-mod-wsgi python-dev`
            * Enable wsgi: `sudo a2enmod wsgi`
            * Create app folder `sudo mkdir -p /var/www/catalog`
            * Change owner of the new folder `sudo chown -R grader:grader catalog`
            * Create wsgi file for the app (catalog.wsgi) inside the catalog folder and add the below code:
                ```
                import sys
                import logging
                logging.basicConfig(stream=sys.stderr)
                sys.path.insert(0, "/var/www/catalog/")
                from catalog import app as application
                application.secret_key = 'secret_key_for_github'
                ```
            * Copy catalog project inside the catalog folder `git clone https://github.com/alexeylyutik/itemcatalog.git`
            * Move catalog project only folder to the current folder `mv -v /var/www/catalog/itemcatalog/catalog/* /var/www/catalog`
            * Rename application name `mv application.py  __init__.py`
            * Create catalog apache2 config under
              ```
              <VirtualHost *:80>
                ServerName 34.220.230.36
                ServerAlias ec2-34-220-230-36.us-west-2.compute.amazonaws.com
                ServerAdmin admin@34.220.230.36
                SSLEngine on
                SSLCertificateFile "/etc/ssl/certs/apache-selfsigned.crt"
                SSLCertificateKeyFile "/etc/ssl/private/apache-selfsigned.key"
                ServerName 34.220.230.36
                ServerAlias ec2-34-220-230-36.us-west-2.compute.amazonaws.com
                ServerAdmin admin@34.220.230.36
                WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
                WSGIProcessGroup catalog
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
              ```
            * Enable SSL `sudo a2enmod ssl`
            * Create certificates `sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/apache-selfsigned.key -out /etc/ssl/certs/apache-selfsigned.crt`
        * Virtual Environment Setup
            * `sudo apt install python-pip`
            * `sudo pip install virtualenv`
            * `sudo virtualenv venv`
            * `source venv/bin/activate`
            * `sudo chmod -R 777 venv`
        * Catalog Project dependencies
            * `sudo pip install Flask`
            * `sudo pip install sqlalchemy`
            * `sudo pip install passlib`
            * `sudo pip install itsdangerous`
            * `sudo pip install flask`
            * `sudo pip install flask-bootstrap`
            * `sudo pip install flask-httpauth`
            * `sudo pip install request`
            * `sudo pip install requests`
            * `sudo pip install oauth2client`
        * PostgreSQL
            * `sudo apt-get install libpq-dev python-dev`
            * `sudo apt-get install postgresql postgresql-contrib`
            * `sudo su - postgres`
            * `psql`
            * `psql`
            * `CREATE USER catalog WITH PASSWORD 'password';`
            * `ALTER USER catalog CREATEDB;`
            * `CREATE DATABASE catalog WITH OWNER catalog;`
            * `\c catalog`
            * `REVOKE ALL ON SCHEMA public FROM public;`
            * `GRANT ALL ON SCHEMA public TO catalog;`
            * Change create engine line in your `__init__.py` and `models.py` to: `engine = create_engine('postgresql://catalog:password@localhost/catalog')`
    * iv. A list of any third-party resources you made use of to complete this project.
        * [Maketecheasier Apache SSL enable](https://www.maketecheasier.com/apache-server-ssl-support/)
        * [Digital Ocean Redirect to HTTPS](https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-apache-in-ubuntu-16-04)
        * [Apache Org](https://httpd.apache.org/docs/2.4/ssl/ssl_howto.html#onlystrong)

3. Locate the SSH key you created for the grader user.
    * Done
4. During the submission process, paste the contents of the grader user's SSH key into the "Notes to Reviewer" field.
    * Done

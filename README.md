# Project Details
-	As part of this project you would create your own Linux Server Instance, secure the server, install required packages and perform required configurations. Deploy a web application at the configured server Instance. The deployed web application should be accessible via a URL.
## Basic Info:
-   IP Address: 13.233.190.226
-   App URL: http://ec2-13-233-190-226.ap-south-1.compute.amazonaws.com/
## Create a new Linux server instance
-   Login to [Amazon Lightsail](https://lightsail.aws.amazon.com/)
-	Under the 'Select a blueprint' chose 'OS Only'
-	Select Ubuntu 16.04 LTS
-	Scroll down to 'Choose your instance plan' and let the default plan be selected, i.e. $3.50 USD (First month free!)
-	Give your instance a name
-	Click on Create Instance
-	Click on the instance created in above step
-	Click on 'Connect using SSH' button
    -   A command prompt would get opened to perform required operations on your server

## Make your Linux Server Secure
-   Update current installed packages:
    -	Run below commands
        -   `sudo apt-get update`
        -   `sudo apt-get upgrade`

-   Connect to Ubuntu from local machine:
    -	Navigate to [SSH Keys](https://lightsail.aws.amazon.com/ls/webapp/account/keys) and download private key
    -	Copy private key content
    -	Open terminal at local machine and write private key to system with below steps:
        -   `mkdir .ssh`
        -   `touch .ssh/auth_key` 
        -   `nano .ssh/auth_key` 
        -   Paste copied private key content, save and exit
        -   `ssh ubuntu@13.233.190.226 -p 22 -i ~/.ssh/auth_key`
        -   Now you are connected to ubuntu instance from local machine 

## Change the SSH port from 22 to 2200
-   Add port 2200 to sshd_config file:
    -   `sudo nano /etc/ssh/sshd_config`
    -   Add a line 'Port 2200' after line 'Port 22' 
        -  Save and Exit
-   Remove port 22 from sshd_config file:
    -   In next few steps, we will configure Uncomplicated Firewall (UFW), and open firewall for port 2200
    -   After that we will remove port 22 from sshd_config file as well as disable it in UFW
    -   This way ensures that we do not lock ourselves out of ubuntu server

## Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
-   Check UFW status (default UFW status is inactive)
    -   `sudo ufw status`  
-   Deny any incoming request by default
    -   `sudo ufw default deny incoming`
-   Allow any outgoing request by default
    -   `sudo ufw default allow outgoing`
-   Allow incoming connection for SSH (port 2200)
    -   `sudo ufw allow 2200/tcp` 
-   Allow incoming connection for HTTP (port 80)
    -   `sudo ufw allow www`
-   Allow incoming connection for NTP (port 123) 
    -   `sudo ufw allow 123/udp`
-   Enable UFW
    -   `sudo ufw enable`
-   Check UFW status (UFW status should be active now)
    -   `sudo ufw status`
-   Exit from server

## Connect to Linux Server Instance from Local Machine
-   Perform below steps at Amazon Lightsail:
    -   Click on ubuntu instance
    -   Click on 'Networking' tab
    -   Under Firewall add below ports by clicking 'Add another':
        -   2200 [Application: Custom; Protocol: TCP; Port range: 2200]
        -   123  [Application: Custom; Protocol: TCP; Port range: 123]
    -   Reboot server instance
    -   Run below command at local machine:
        -   `ssh ubuntu@13.233.190.226 -p 2200 -i ~/.ssh/auth_key`

## Disable port 22 for SSH connection
-   Update sshd_config file:
    -   `sudo nano /etc/ssh/sshd_config`
    -   Remove line 'Port 22'
    -   Save and Exit
-   Disable port 22 in UFW:
    -   `sudo ufw deny 22`
-   Perform below steps at Amazon Lightsail:
    -   Click on ubuntu instance
    -   Click on 'Networking' tab
    -   Under Firewall remove port 22 with option listed on click of 'Edit rules'

## Give grader access
-   Create a new user account named grader
    -   `sudo adduser grader`
-   Give grader the permission to sudo
    -   Add and write grader file under sudoers directory
        -   `sudo touch /etc/sudoers.d/grader`
        -   `sudo nano /etc/sudoers.d/grader`
        -   Write `grader ALL=(ALL:ALL) ALL` in grader file
        -   Save and exit
-   Create an SSH key pair for grader using the ssh-keygen tool
    -   Perform below steps at local machine:
        -   `ssh-keygen`
            -   Enter file in which to save the key(for windows OS) - `/c/Users/<user-name>/.ssh/my_key.rsa`
        -   Print the public key usnig `cat ~/.ssh/my_key.rsa.pub`
        -   Copy the public key
    -   Perform below steps at Linux Ubuntu Instance:
        -   `cd /home/grader`
        -   `sudo mkdir .ssh`
        -   `sudo touch .ssh/authorized_keys`
        -   `sudo nano .ssh/authorized_keys`
        -   Paste the copied public key, save and exit        
        -   `sudo chmod 700 /home/grader/.ssh` 
        -   `sudo chmod 644 /home/grader/.ssh/authorized_keys`
        -   `sudo chown grader .ssh`
        -   Reboot server instance

## Prepare to deploy the Web Application at Server
-   Login as grader
    -   `ssh grader@-p 2200 -i ~/.ssh/my_key.rsa`
-   Disable SSH for root
    -   `sudo nano /etc/ssh/sshd_config`
    -   Change "PermitRootLogin" to "no"
    -   Save and Exit
    -   `sudo service ssh restart`
-   Configure the local timezone to UTC
    -   `sudo dpkg-reconfigure tzdata`    
-   Install Apache
    -   `sudo apt-get install apache2`
-   Install mod_wsgi
    -   `sudo apt-get install libapache2-mod-wsgi`  
-   Install git
    -   `sudo apt-get install git`  
-   Restart apache server 
    -   `sudo apache2ctl restart`  
-   Open `13.233.190.226`(Public IP) in web browser
    -   "Apache2 Ubuntu Default Page" would get opened

## Deploy the Web Application at Server
-   Login as grader
    -   `ssh grader@13.233.190.226 -p 2200 -i ~/.ssh/my_key.rsa`
-   Clone Item Catalog Web Application from GitHub:
    -   `cd /var/www`
    -   `sudo mkdir catalog`
    -   `sudo chown -R grader:grader catalog`
    -   `cd catalog`
    -   `git clone https://github.com/samrat-b/item-catalog.git catalog`
    -   `sudo touch catalog.wsgi` 
    -   `sudo nano catalog.wsgi`
    -   Insert below code into the above created file:
    -   ```
        import sys
        import logging
        logging.basicConfig(stream=sys.stderr)
        sys.path.insert(0, "/var/www/catalog/")

        from catalog import app as application
        application.secret_key = 'supersecretkey'  
        ```
    -   Rename `project.py` to `__init__.py`             
-   Install and Activate VM:
    -   `sudo apt-get install python-pip`
    -   `sudo pip install virtualenv`
    -   `sudo virtualenv venv`
    -   `source venv/bin/activate`
    -   `sudo chmod -R 777 venv`  
-   Install required packages and make required changes:
    -   `pip install Flask`
    -   `pip install httplib2 oauth2client sqlalchemy requests psslib`
    -   `pip install psycopg2`
    -   `cd /var/www/catalog/catalog`
    -   `nano __init__.py`
        -   Replace `client_secrets.json` with `/var/www/catalog/catalog/client_secrets.json`
        -   Replace the host with your Amazon Lightsail public IP address and port with 80
            -   `app.run(host='0.0.0.0', port=5000)` => `app.run(host='13.233.190.226', port=80)`
-   Configure and enable Virtual Host:
    -   `sudo touch /etc/apache2/sites-available/catalog.conf`
    -   `sudo nano /etc/apache2/sites-available/catalog.conf`
    -   Copy below code, paste in catalog.conf and save:
        ```
        <VirtualHost *:80>
        ServerName 13.233.190.226
        ServerAlias ec2-13-233-190-226.ap-south-1.compute.amazonaws.com
        ServerAdmin admin@13.233.190.226
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
-   Setup DB:                       
    -   `sudo apt-get install libpq-dev python-dev`
    -   `sudo apt-get install postgresql postgresql-contrib`
    -   `sudo su - postgres`
    -   `psql`
    -   SQL Operations:
        -   `CREATE USER catalog WITH PASSWORD 'demopass';`
        -   `ALTER USER catalog CREATEDB;`
        -   `CREATE DATABASE catalog WITH OWNER catalog;`
        -   `\c catalog`
        -   `REVOKE ALL ON SCHEMA public FROM public;`
        -   `GRANT ALL ON SCHEMA public TO catalog;`
        -   `\q`
        -   `exit`
    -   `cd /var/www/catalog/catalog`
    -   `sudo nano dbsetup.py`
        -   Update engine:
            -   `engine = create_engine('postgresql://catalog:demopass@localhost/catalog')`
            -   Save and exit
    -   `sudo nano __init__.py`
        -   Update engine:
            -   `engine = create_engine('postgresql://catalog:demopass@localhost/catalog')`
            -   Save and exit
    -   `sudo nano create_categories.py`
        -   Update engine:
            -   `engine = create_engine('postgresql://catalog:demopass@localhost/catalog')`
            -   Save and exit
    -   `python /var/www/catalog/catalog/dbsetup.py` 
    -   `python /var/www/catalog/catalog/create_categories.py`           
-   Update Authorized Origins and Authorized redirect URIs
    -   Go to Google Developer Console and add `ec2-13-235-16-45.ap-south-1.compute.amazonaws.com` under 'Authorized JavaScript origins' and under 'Authorized redirect URIs'
    -   Update `/var/www/catalog/catalog/client_secrets.json` at Server
-   Run the deployed application:
    -   `sudo service apache2 restart`
    -   Enter `ec2-13-235-16-45.ap-south-1.compute.amazonaws.com` in Web Browser

### References:
https://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt/138442; https://github.com/iliketomatoes/linux_server_configuration; https://github.com/chuanqin3/udacity-linux-configuration

# Project Details
-	….
-	….

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

### Update current installed packages:
-	Run below commands
    -   `sudo apt-get update`
    -   `sudo apt-get upgrade`

### Connect to Ubuntu from local machine:
-	Navigate to [SSH Keys](https://lightsail.aws.amazon.com/ls/webapp/account/keys) and download private key
-	Copy private key content
-	Open terminal at local machine and write private key to system with below steps:
    -   `mkdir .ssh`
    -   `touch .ssh/auth_key` 
    -   `nano .ssh/auth_key` 
    -   Paste copied private key content, save and exit
    -   `ssh ubuntu@13.232.190.117 -p 22 -i ~/.ssh/auth_key`
    -   Now you are connected to ubuntu instance from local machine 

## Change the SSH port from 22 to 2200

### Add port 2200 to sshd_config file:
-   `sudo nano /etc/ssh/sshd_config`
-   Add a line 'Port 2200' after line 'Port 22' 
    -  Save and Exit

### Remove port 22 from sshd_config file:
-   In next few steps, we will configure Uncomplicated Firewall (UFW), and open firewall for port 2200
-   After that we will remove port 22 from sshd_config file as well as disable it in UFW
-   This way ensures that we do not lock ourself out of ubuntu server

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
-   Exit from server and connect from local machine using below command:
    -   `ssh ubuntu@13.232.190.117 -p 2200 -i ~/.ssh/auth_key`

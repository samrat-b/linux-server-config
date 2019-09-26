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
        -   `ssh ubuntu@13.232.190.117 -p 22 -i ~/.ssh/auth_key`
        -   Now you are connected to ubuntu instance from local machine 

## Change the SSH port from 22 to 2200
-   Add port 2200 to sshd_config file:
    -   `sudo nano /etc/ssh/sshd_config`
    -   Add a line 'Port 2200' after line 'Port 22' 
        -  Save and Exit
-   Remove port 22 from sshd_config file:
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
-   Exit from server

## Connect to Linux Server Instance from Local Machine
-   Perform below steps at Amazon Lightsail:
    -   Click 'Manage' listed after clicking three vertical dots on your server instance
    -   Click on 'Networking' tab
    -   Under Firewall add below ports by clicking 'Add another':
        -   2200 [Application: Custom; Protocol: TCP; Port range: 2200]
        -   123  [Application: Custom; Protocol: TCP; Port range: 123]
    -   Reboot server instance
    -   Run below command at local machine:
        -   `ssh ubuntu@13.232.190.117 -p 2200 -i ~/.ssh/auth_key`

## Disable port 22 for SSH connection
-   Update sshd_config file:
    -   `sudo nano /etc/ssh/sshd_config`
    -   Remove line 'Port 22'
    -   Save and Exit
-   Disable port 22 in UFW:
    -   `sudo ufw deny 22`
-   Perform below steps at Amazon Lightsail:
    -   Click 'Manage' listed after clicking three vertical dots on your server instance
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
            -   Enter file in which to save the key(for windows OS) - `/c/Users/<user-name>/.ssh/grade_key`
        -   Print the public key usnig `cat ~/.ssh/grade_key.pub`
        -   Copy the public key
    -   Perform below steps at Linux Ubuntu Instance:
        -   `touch .ssh/grader_key`
        -   `nano .ssh/grader_key`
        -   Paste the copied public key, save and exit
        -   `sudo chmod 700 /home/grader/.ssh` 
        -   `sudo chmod 644 /home/grader/.ssh/grader_key`

## Prepare to deploy your project
-   Configure the local timezone to UTC
    -   Login as grader: `ssh ubuntu@13.232.190.117 -p 2200 -i ~/.ssh/grader_key`
    -   Chnage timezone to UTC: `sudo dpkg-reconfigure tzdata`
-   Install and configure Apache to serve a Python mod_wsgi application.
    >> Continue from here...    

### References:
https://github.com/chuanqin3/udacity-linux-configuration
https://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt/138442

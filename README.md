# Preparing Debian 11 for Ansible
This repository contains information to prepare a clean minimal Debian 11 target system for use with Ansible. 
The script assumes that a user named "ashish" has already been created on the system.

## STEP 1: Prepare target machine

Copy the contents of the script below into init.sh and save it to a file on your Debian 11 system. Then, run the script as the root user.

````sh
#!/bin/bash

# Set the user name
user=ashish

# Update the system
apt update -y && apt upgrade -y

# Install sudo
apt install sudo -y
/usr/sbin/usermod -aG sudo $user

# Add user to sudoers
echo "$user ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

# Install Git
apt install git -y

# Install Python
apt install python3 -y

# Clone the GitHub repository
cd ~
git clone https://github.com/ashishsalve/ansible-bootstrap.git

# Run the init script
cd ~/ansible-bootstrap
./init.sh


````

Make this script executable and run it.
````sh
chmod +x init.sh
./init.sh
````

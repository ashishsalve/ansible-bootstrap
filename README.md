# Preparing Debian 11 for Ansible
This repository contains a shell script that will prepare a clean minimal Debian 11 system for use with Ansible. 
The script assumes that a user named "ashish" has already been created on the system.

## Running the script

To run the script, copy the contents of the script below and save it to a file on your Debian 11 system. Then, run the script as the root user.

````sh
#!/bin/bash

# Update the system
apt update && apt upgrade -y

# Install sudo
apt install sudo -y
/sbin/usermod -aG sudo ashish

# Add user ashish to sudoers
user=ashish
echo "$user ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

# Install Git
apt install git -y

$ Install Python
apt install python3 -y

# Clone the GitHub repository
git clone https://github.com/ashishsalve/ansible-bootstrap.git

# Run the init script
cd ansible-bootstrap
./init.sh

````

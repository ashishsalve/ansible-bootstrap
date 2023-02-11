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

echo "Auto configuration setup complete. This machine is now ready for Ansible deployment."

````

Make this script executable and run it.
````sh
chmod +x init.sh
./init.sh
````

# Control Machine
Follow these instructions on control machine to auto configure target machines.

## SSH Auto-login
````sh
# Add public key for public key authentication
ssh-copy-id ashish@hostname

# Login via SSH
ssh ashish@hostname

# become root
su
````
Follow steps mentioned at the top of this page to create and run init.sh
Exit the machine once init.sh completes execution.

# Ansible Control Machine
Assuming ansible is installed on control machine, use following to create secrets vault and run the playbook:
````sh
ansible-vault create secrets.yml
# OR to edit vault, use
# ansible-vault edit secrets.yml
ansible-playbook -i 192.168.0.121, configure.yml --user ashish --ask-vault-pass
````

Here is an example playbook that sets up vault secrets, apply those secrets as environment variables on target machine and install apache.
````secrets.yml
vault_TANGO: CHARLIE
vault_REDIS_KEY: 'ewogICAiSW5zdXJhbmNlQ29tcGFuaWVzIjp7CiAgICAgICJUaW1lIjoiTWF5IDIwMjEiLAogICAgICAiVG9wIEluc3VyYW5jZSBDb21wYW5pZXMiOlsKICAgICAgICAgewogICAgICAgICAgICAiTm8iOiIxIiwKICAgICAgICAgICAgIk5hbWUiOiJCZXJrc2hpcmUgSGF0aGF3YXkgKCBCUksuQSkiLAogICAgICAgICAgICAiTWFya2V0IENhcGl0YWxpemF0aW9uIjoiJDY1NSBiaWxsaW9uIgogICAgICAgICB9CiAgICAgIF0sCiAgICAgICJzb3VyY2UiOiJpbnZlc3RvcGVkaWEuY29tIiwKICAgICAgInVybCI6Imh0dHBzOi8vd3d3LmludmVzdG9wZWRpYS5jb20vYXJ0aWNsZXMvYWN0aXZlLXRyYWRpbmcvMTExMzE0L3RvcC0xMC1pbnN1cmFuY2UtY29tcGFuaWVzLW1ldHJpY3MuYXNwIgogICB9Cn0='
````

````configure.yml
---
- name: Install Apache on Debian 11
  hosts: all
  become: yes
  vars:
    user: ashish
  vars_files: 
    - secrets.yml

  tasks:
    - name: Update package index
      apt:
        update_cache: yes

    - name: Install Apache
      apt:
        name: apache2
        state: present

    - name: Start Apache service
      service:
        name: apache2
        state: started
        enabled: yes

    - name: Create environment variables
      shell: |
        echo "export TANGO={{ vault_TANGO }}" >> /home/{{ user }}/.bashrc
        echo "export REDIS_KEY='{{ vault_REDIS_KEY }}'" >> /home/{{ user }}/.bashrc

    - name: Display environment variables
      become_user: "{{ user }}"
      shell: |
        echo "TANGO is set to: $TANGO"
        echo "REDIS_KEY is set to: $REDIS_KEY"




````


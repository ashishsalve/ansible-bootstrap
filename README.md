# Preparing Debian 11 for Ansible
This repository contains information to prepare a clean minimal Debian 11 target system for use with Ansible. 
The script assumes that a user named "user" has already been created on the system.

# Control Machine
Follow these instructions on control machine:

## SSH Auto-login
````sh
# Add public key for public key authentication
ssh-copy-id user@hostname

# Login via SSH
ssh user@hostname

# become root
su
````


## STEP 1: Prepare target machine

Copy the contents of the script below into init.sh and save it to a file on your Debian 11 system. Then, run the script as the root user.

````sh
#!/bin/bash

# Set the user name (retrieve name of the user you became root from)
user=${SUDO_USER:-$USER}


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


Follow steps mentioned at the top of this page to create and run init.sh
Exit the machine once init.sh completes execution.

# Ansible Control Machine
Assuming ansible is installed on control machine, use following to create secrets vault and run the playbook:
````sh
ansible-vault create secrets.yml
# OR to edit vault, use
# ansible-vault edit secrets.yml
ansible-playbook -i hostname, configure.yml --user ashish --ask-vault-pass
````

Here is an example playbook that sets up vault secrets, apply those secrets as environment variables on target machine and install apache.
````secrets.yml
vault_TANGO: 'CHARLIE'
vault_REDIS_KEY: 'SOME string here....'
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

## Decrypt base64 environment variables
Following example shows how to decode and export base64 encoded environment variable into file
````sh
#!/bin/bash

# Decode the base64 value of the JSON
decoded_redis_key=$(echo "$REDIS_KEY" | base64 --decode)

# Write the decoded JSON to a file
echo "$decoded_redis_key" > output.json
````

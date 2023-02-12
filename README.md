# Preparing Debian 11 for Ansible
This repository contains information and scripts for preparing a clean, minimal Debian 11 target system for use with Ansible. The script assumes that a user named "user" has already been created on the system.



# Control Machine
To prepare your control machine, follow these steps:


## SSH Auto-login

To set up SSH auto-login, perform the following actions:
````sh
# Add public key for public key authentication
ssh-copy-id user@hostname

# Login via SSH
ssh user@hostname

# become root
su
````


## Target Machine Preparation

1. Copy the contents of the script below into a file named `init.sh`

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

2. Make the script executable by running `chmod +x init.sh`.
3. Run the script by executing `./init.sh`.

## Ansible Control Machine

Follow steps mentioned at the top of this page to create and run init.sh
Exit the machine once init.sh completes execution.

# Ansible Control Machine
Assuming Ansible is installed on the control machine, perform the following steps to create a secrets vault and run the playbook:

````sh
ansible-vault create secrets.yml
# OR to edit vault, use
# ansible-vault edit secrets.yml
ansible-playbook -i hostname, configure.yml --user ashish --ask-vault-pass
````


This is an example of decrypted vault secrets file:
````secrets.yml
vault_TANGO: 'CHARLIE'
vault_REDIS_KEY: 'SOME string here....'
````

# Workfolow
For all projects, follow these steps:
1. Create a private repository and copy the `configure.yml` file from this repository.
2. Create an init.sh file.
3. Create a `service-utils/` directory containing any utilities or scripts that are not part of the primary codebase.
4. Run the playbook, SSH into the target machine, and run `init.sh`. With these steps, the server should be ready to proceed with development.


````configure.yml
---
- name: Setup debian11 machine secrets for grantsfire API
  hosts: all
  become: yes
  vars_files: 
    - secrets.yml

  tasks:
    - name: Update package index
      apt:
        update_cache: yes

    - name: Set ansible_user variable
      set_fact:
        ansible_user: "{{ ansible_ssh_user }}"

    - name: Create environment variables
      shell: |
        echo "export DB_CONNECTION={{ vault_DB_CONNECTION }}" >> /home/{{ ansible_user }}/.bashrc
        

    - name: Copy init.sh file to home directory
      copy:
        src: init.sh
        dest: /home/{{ ansible_user }}/init.sh
      become: yes

    - name: Make init.sh executable
      file:
        path: /home/{{ ansible_user }}/init.sh
        mode: 0755
      become: yes

    - name: Set ansible_user variable
      set_fact:
        ansible_user: "{{ ansible_ssh_user }}"

    - name: Copy service-utils folder
      copy:
        src: service-utils/
        dest: /home/{{ ansible_user }}/service-utils/
      become: yes
      become_user: "{{ ansible_user }}"




````

## Storing JSON files for service accounts
To store JSON files for service accounts, make sure to convert the JSON object to a base64 string and assign the string value to a variable. Then, decrypt it on the target machine.

# DVWA-Ansible
Easy way to install DVWA on Ubuntu 19.04 using Ansible.
For more information on this app please see the upstream repo https://github.com/ethicalhack3r/DVWA

# What does this playbook do?
It installs a LAMP stack(apache,mysql and php) and configures them to get a working system to run DVWA

# Credit
Credit goes to https://github.com/appsecco/vulnerable-apps which this playbook is based on.
I have fixed things that ansible has been complaining about so you can use it with the lastest version and to get it to clone from git master instead of using a zip file release

# Running playbook
Setup an Ubuntu 19.04 server with a user called ansible and update the IP in the hosts file and do the following from the root of this folder on a host that has ansible installed

ansible-playbook main.yml -i hosts --ask-become-pass -v

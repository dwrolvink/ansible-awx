# ansible-awx

- [Windows modules](https://docs.ansible.com/ansible/latest/modules/list_of_windows_modules.html)
- [Ansible - Windows Usage](https://docs.ansible.com/ansible/latest/user_guide/windows_usage.html)

# Installing AWX
- Install Centos 7 with GUI
- [Install AWX](https://www.howtoforge.com/tutorial/how-to-install-ansible-awx-with-docker-on-centos/) or [Install script](https://gist.github.com/dasgoll/7073664f0f7f73f4aa3e7cf8c95a8dbc)

---

# Managing Windows hosts with AWX
## Useful commands and other information
```bash
# Login to awx-task docker
docker ps
docker exec -it <docker id> /bin/bash

# check kerberos working
kinit user@DOMAIN.NAME
klist

# Reload AWX
ansible-playbook install.yml -i inventory

# Location installation config
~/awx/installer/inventory

# Quick test by running module directly
ansible windows -m win_ping --ask-vault-pass

# Quick test by running playbook
ansible-playbook <playbookname> --limit <group name in default inventory> --ask-vault-pass 
```

## Getting kerberos authentication working
1. Before installing AWX, set the correct value for domain name, and dns servers in the installation config.
2. After installing AWX, login to awx-task docker container, and update /etc/krb.conf to these specifications: [krb.conf](https://docs.ansible.com/ansible-tower/latest/html/administration/kerberos_auth.html)
3. In the same container, check if you can ping the windows machine using the dns name
4. In the same container, check if kerberos is working (see code block above).

## Getting AWX to win_ping windows hosts
1. Make a Windows group (windows_hosts) in your inventory of choice
2. Add your windows hosts to that group, and add the following vars:
  ```yaml
  ---
  ansible_connection: winrm
  ansible_winrm_transport: kerberos
  ansible_winrm_server_cert_validation: ignore
  ```
3. Create a new Machine credential, where username is user@DOMAIN.NAME (AD admin)
4. Add a new playbook to your project (hello_world_windows.yml):  
    - [hello_world_windows.yml](https://raw.githubusercontent.com/dwrolvink/ansible-awx/master/hello_world_windows.yml)
    > Note: I keep adding extra 'hello worlds' to this file, like win_copy. If you only want to test win_ping, only keep the first task. Also be sure to edit any data in there to match your environment.
5. Create a new template using your inventory of choice, the project your playbook is in, select your playbook (reload your project if you just added the playbook and it doesn't show up yet), and select your windows credential.

## Run your playbook
That's it, run your playbook, and it should work!. For troubelshooting: be sure to have a close look at the /etc/krb.conf file **in your awx-task container**, and do low level troubleshooting in there too. Make sure you have the right capitalization in your krb.conf file, and your machine credential. You can reload awx by running the installer again (see useful commands above). Make sure the installer doesn't overwrite any changes you made. (DNS settings are set in the awx/installer/inventory file, and the DNS servers don't appear in /etc/resolv.conf on the container (not sure where that's set), though in that file, you should see your domain name listed.

# Elevation and installation of applications
## Setup Credential to use correct elevation method
1. Go to your AD credential in AWX
2. Set `Priviledge Escalation Method` to runas
3. Set the `Priviledge Escalation Username` and `Priviledge Escalation Password` (the same as the normal user in my case)

## Create the installation playbook
In my example, we'll be installing notepad++ on the windows host using win_chocolatey

1. Add the following file to your git project:
  - [install_notepadplusplus.yml](https://raw.githubusercontent.com/dwrolvink/ansible-awx/master/Install_Notepadplusplus.yml)
2. Update your project in AWX to get the new file
3. Copy the previous template, and change the name to 'Install Notepad++'
4. Change the playbook to 'install_notepadplusplus.yml'
5. Under Options, check `Enable Priviledge Escalation`
6. Run your template.

# Working with Active Directory and further exploration
Check the [Windows modules](https://docs.ansible.com/ansible/latest/modules/list_of_windows_modules.html) list for hints of what is possible. 

For example, you have `win_domain_group` and `win_domain_user` modules, in the following playbook I use those to create a user and a group in a pre-existing OU (and make the user member of the group):
- [hello_world_ad.yml](https://github.com/dwrolvink/ansible-awx/blob/master/hello_active_directory.yml)

# Deploy Windows VM in Vsphere
Used:
- Windows2016 template
- Template asks for ip, netmask, gateway
- ESXI 6.7
- Using a VSphere server

1. Login to your AWX server and login to the awx-task docker container: install Pyvmomi using pip
```bash
docker ps #pick name of awx-task
docker exec -it [name] /bin/bash
pip install Pyvmomi
exit
```

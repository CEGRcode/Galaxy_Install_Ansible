
To run the ansible playbook ssh to machine you intend to run Galaxy on. The guideline assumes you login as `ubuntu` user. The code is tested for ubuntu 20.04 LTS

1- login to Galaxy machine as `ubuntu` user and install `Ansible` for running Anisble playbooks and `sshpass` to allow ssh to Pulsar machine using Ansible playbook

```
apt install ansible

apt-get install sshpass
```

2- on Pulsar machine, login as as `ubuntu` user, create password for `ubuntu` user, and change the ssh config file to allow login by using password.

```
sudo passwd ubuntu
```

`sudo vi /etc/ssh/sshd_config` inside the file, change `PasswordAuthentication no` To `PasswordAuthentication yes`
    
 3- Restart ssh service to reflect the changes made in the config file
 
 ```
 sudo service ssh restart
 ```

4- On Galaxy machine as `ubuntu` user, clone the Ansible playbooks
  
  ```
  git clone https://github.com/CEGRcode/Galaxy_Install_Ansible.git`
    
  cd Galaxy_Install_Ansible
  ```
5- Make sure the hostname of Galaxy machine is correct and IP address of Pulsar machine is correct/

`vi hosts` update the hostname as your Galaxy machine hostname: For example the host name on Ansible is now "galaxy.dev.cac.cornell.edu" change it to your own hostname. The IP address of Pulsar machine is "128.84.9.67"
   
 
6- on `Galaxy` machine as `ubuntu` user Run Ansible playbook to deploy `Galaxy` (~20min).
 
```
ansible-playbook galaxy.yml
```

7-On `Galaxy` machine as `ubuntu` user Run Ansible playbook to deploy `Pulsar` on remote machine for which the IP is provided in Step 5. Ansible will ssh to Pulsar machine and will install Pulsar. When prompted for password type passwrod you created in Step 2.
    
```
ansible-playbook pulsar.yml --ask-pass
```

Notes

A) In the the groupvars/all.yml file, a password is used for secure communication between pulsar and Galaxy. Change it if you like.


Current Ansible includes

Galaxy with
  CVMFS
  systemd
  slurm
  nginx
  SSL
  RabbitMQ

Pulsar with
  CVMFS
  systemd
  slurm

RabbitMQ is used for communication between Galaxy and Pulsar


Note 1:
number of CPUs in Pulsar machine needs to be updated based on specific machine configuration at:
group_vars/pulsarservers.yml

Galaxy root directory, where all the files related to Galaxy will be installed, can be changed in group_vars/galaxyservers.yml


Note 2:
 Currently there is a bug in Galaxy that Admin->Manage Allowlist cannot be updated. To overcome this issue, you can follow these steps:
A. `sudo touch /sr/galaxy/server/config/sanitize_allowlist.txt`
B. `sudo chmod 777 /sr/galaxy/server/config/sanitize_allowlist.txt`

Note 3:

Ansible playbook currently copies EGC version of tool_conf.xml and welcome.html file.
The following files are changed in ansible-playbook

A. In `templates/nginx/galaxy.j2`  `welcome.html.sample` is changed to `welcome.html`
B. In `/roles/galaxyproject.galaxy/defaults/main.yml` `tool_conf.xml.sample` is changed to `tool_conf.xml`

Note 4: 
In the current git repository roles are downloaded for galaxy 20.09. Should a new version of Galaxy be installed, these roles and their versions needs to be updated by

`ansible-galaxy install -p roles -r requirements.yml`

If the roles ar updated please update changes in Note 3 as well.
 
Note 5: 

In this ansible, the default admin for the deployed galaxy is brc_epigenomics@cornell.edu

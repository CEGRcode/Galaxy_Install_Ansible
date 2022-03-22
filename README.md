<h1> About</h1>
Ansible playbooks and instructions for deployment of Galaxy and Pulsar. Wiki pages of this repository include instructions for spinning up multiple volumes VM for Galaxy on openstack,  installing local reference genomes on Galaxy, and uploading and downloading workflows from Galaxy.

<h2> Ansible Playbook Features </h2>

The current Ansible playbook for Galaxy deploys `Galaxy base`, `CVMFS`, `systemd`, `slurm`, `nginx`, `SSL` and `RabbitMQ`. The current pulsar playbook deploys `Pulsar base`,`CVMFS`, `systemd`, and `slurm`.
  

<h2> Deploy Galaxy and Pulsar using Ansible </h2>

The playbooks are tested for ubuntu 20.04 LTS clean machines for Galaxy and Pulsar.

1- Login to Galaxy machine as `ubuntu` user and install `Ansible` for running Anisble playbooks and `sshpass` to allow ssh to Pulsar machine using Ansible playbook

```
apt install ansible

apt-get install sshpass
```

2- Login to Pulsar machine as as `ubuntu` user, create password for `ubuntu` user, and change the ssh config file to allow login by using password.

```
sudo passwd ubuntu
```

`sudo vi /etc/ssh/sshd_config` inside the file, change `PasswordAuthentication no` To `PasswordAuthentication yes`
    
 3- Restart ssh service to reflect the changes made in the config file
 
 ```
 sudo service ssh restart
 ```
 You may want to logout and ssh as ubuntu user with password to Pulsar machine to make sure your changes are workig fine.

4- On Galaxy machine as `ubuntu` user, clone the Ansible playbooks
  
  ```
  git clone https://github.com/CEGRcode/Galaxy_Install_Ansible.git`
    
  cd Galaxy_Install_Ansible
  ```
5- Make sure the hostname of Galaxy machine is correct and IP address of Pulsar machine is correct. By following 5.1 and 5.2 steps:

  5-1 Update the hostname of your Galaxy machine and IP address of Pulsar machine. 
  ```
  vi hosts
  ```
  The hostname on Ansible is now "galaxy.dev.cac.cornell.edu" change it to your own hostname. 
  IP address of Pulsar machine is "128.84.9.67". Change it to your Pulsar machine IP address.  Please note Pulsar machine does not need a hostname.
  
  5-2 In the file for Pulsar varaibles we should also provide the hostname of Galaxy machine. So, update the hostname of Galaxy at:
  ```
  vi group_vars/pulsarservers.yml
  ```
 Change `galaxy_server_url: galaxy.dev.cac.cornell.edu` to `galaxy_server_url: your_galaxy_machine_hostname`. Please note that although the naming is called galaxy_server_url, Galaxy url shouldn't be used and it is just a poor naming.
   
 
6- on `Galaxy` machine as `ubuntu` user Run Ansible playbook to deploy `Galaxy` (~20min).
 
```
ansible-playbook galaxy.yml
```

7- On `Galaxy` machine as `ubuntu` user ssh to Pulsar machine with password. This will allow you to answer to the questions of First Time Connections to a remote machine. 
```
ssh ubuntu@IP_address_pulsar_machine
```
Answer yes to the following questions
`Are you sure you want to continue connecting (yes/no/[fingerprint])?`


8-On `Galaxy` machine as `ubuntu` user Run Ansible playbook to deploy `Pulsar` on remote machine for which the IP address is provided in Step 5. Ansible will ssh to Pulsar machine and will deploy Pulsar. When prompted for password type passwrod you created in Step 2 (~15min)
    
```
ansible-playbook pulsar.yml --ask-pass
```
<h2> Notes on Running Galaxy and Pulsar Playbooks </h2>

**Note 1:**

Number of CPUs in Pulsar machine needs to be updated based on specific machine configuration at `group_vars/pulsarservers.yml`
`CPUs: 12` . Change it based on your machine configurations. Leave couple of CPUs for other tasks.

Currently Pulsar only run Mapping step of pipeline. In the current setting 4 CPUs is used for each mapping job in the following file. Please change it based on your Pulsar machine specfications. 

under `destinations`, `destination`, inside `param id="submit_native_specification"` it is set to ` --nodes=1 --ntasks=1 --cpus-per-task=4 `

**Note 2:**

Galaxy root directory, where all the files related to Galaxy will be installed, can be changed in `group_vars/galaxyservers.yml`
This is important if you wish to keep galaxy and all its depenencies on different volume.
The following variables are updated in `group_vars/galaxyservers.yml` to install Galaxy on separate volume.

`galaxy_root: /mnt/mountpoint/srv/galaxy`

`file_path: /mnt/mountpoint/data` under galaxy_config->galaxy 

`postgresql_backup_dir: /mnt/mountpoint/data/backups` 


**Note 3:**

Currently there is a bug in Galaxy that Admin->Manage Allowlist cannot be updated. To overcome this issue, you can follow these steps:
 
```
sudo touch /sr/galaxy/server/config/sanitize_allowlist.txt
sudo chmod 777 /sr/galaxy/server/config/sanitize_allowlist.txt
```

**Note 4:**

Ansible playbook currently copies EGC version of tool_conf.xml and welcome.html file. The following files are changed in ansible-playbook

A. In `templates/nginx/galaxy.j2`  `welcome.html.sample` is changed to `welcome.html`

B. In `/roles/galaxyproject.galaxy/defaults/main.yml` `tool_conf.xml.sample` is changed to `tool_conf.xml`

**Note 5:** 

In the current git repository roles are downloaded for galaxy 20.09. Should a new version of Galaxy be installed, these roles and their versions needs to be updated by

```
ansible-galaxy install -p roles -r requirements.yml
```

If the roles ar updated please update changes in Note 3 as well.
 
**Note 6:** 

In this ansible, the default admin for the deployed galaxy is brc_epigenomics@cornell.edu

**Note 7:**

In the the `groupvars/all.yml` file, a password is used for secure communication between pulsar and Galaxy. Change it if you like.

**Note 8:**

This Ansible playbooks are based on Galaxy workshop on 2021 and the versions of Galaxy is `20.09` and Pulsar is `1.0.6`. In the Galaxy workshop on March 2022 the Galaxy version is updatd to `22.01` and Pulsar to `1.0.8`. Updates are shown in `updates2022` branch of this repository, but haven't been tested



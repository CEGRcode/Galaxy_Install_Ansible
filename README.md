
To run the ansible playbook ssh to machine you intend to run Galaxy on. The guideline assumes you login as "ubuntu" user. The code is tested for ubuntu 20.04 LTS

1- on Galaxy machine:

    `apt install ansible`

    `apt-get install sshpass`

2- on Pulsar machine:

    `sudo passwd ubuntu`

    `sudo vi /etc/ssh/sshd_config`

    change `PasswordAuthentication no` To `PasswordAuthentication yes`
    `sudo service ssh restart`

3- on Galaxy machine as ubuntu user:
  
    `git clone https://github.com/CEGRcode/Galaxy_Install_Ansible.git`
    
    `cd Galaxy_Install_Ansible`
    

4- Make sure the hostname is correct in ansible

   `vi hosts` update the hostname as your Galaxy machine hostname: For example the host name on Ansible is now `galaxy.dev.cac.cornell.edu` change it to your own

 
    `ansible-playbook galaxy.yml`

    `ansible-playbook pulsar.yml --ask-pass`


Please note, in the the groupvars/all.yml file, a password needs to be used for secure communication between pulsar and Galaxy.
If Pulsar is installed already, you can just ignore all the pulsar steps.

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

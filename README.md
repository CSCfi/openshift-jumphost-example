# Example of creating a jumphost in OpenStack cPouta
A template on how to create a JumpHost in Openstack with 4 instances behind it.


## Pre-requisites
- Ansible Playbook tested on macOS Ventura and Ubuntu 22.04:  
  - Ansible version 2.15.4
  - Ansible openstack.cloud 2.1.0
  - Python module openstacksdk 1.3.1

For Ubuntu, add the Ansible repo to have the latest version:
```sh
sudo apt-add-repository ppa:ansible/ansible
```

You must have the `openstack.cloud` collection installed: https://docs.ansible.com/ansible/latest/collections/openstack/cloud/index.html  
Type this command:
```sh
ansible-galaxy collection install openstack.cloud
```
and the latest version of `openstacksdk` for Python: https://pypi.org/project/openstacksdk/  
```sh
pip install openstacksdk
```

You must source your project before launching this playbook:  
```sh
source project_xxxxxxx
```

Also, there will be a need to ssh to the newly created instances during the playbook. Create a folder `files` and copy the key public and private key in this folder and add the private key with this command:  
```sh
ssh-add files/id_rsa
```
For this template, you can create a new pair of keys. Just remember to upload it in your cPouta project.  

Create a folder `keys`, add all the public keys you want to authorize to connect to the different hosts.    
During the playbook, it will read those files and add the public keys to the `authorized_keys` file of the different hosts.


## Custom your playbook
In `group_vars/all.yaml`:
- `number_instances`: You can edit the number of instances you want behind the `jumphost`. In this example, we set to `4`.  
- `jumphost_name` & `instances_name`: You can edit the name of the jumphost and the name of the instances.
- `os_image`: You can edit which Operating System you want to use for the jumphost and the instances. (`openstack images list` to retrieve all images available).
- `user`: By default `ubuntu`. If you want CentOS machines, replace it by `cloud-user`.
- `flavor`: Choose the flavor you want to use (`openstack flavor list` to retrieve all flavors available).
- `key_name`: Either the name of your ssh key that you use to connect to cPouta instances or the name of the newly created key for this deployment. (Don't forget to upload this newly created key in your cPouta project).
- `internal_ips`: Edit the list and secure your instances by allowing only few networks to connect through SSH.


## Running the playbook
Run the playbook by typing this command:  
```sh
ansible-playbook main.yaml
```
It will prompt for the network. Use the command `openstack network list` to retrieve this information. Usually it's the name of your project (`project_xxxx`)

Alternatively, you can use this command for running the playbook:
```sh
ansible-playbook -e network=project_xxxx main.yaml
```


## Final steps
After running this playbook, you need to edit the ssh config file. Go to `~/.ssh` and edit `config`.  
Add the jumphost and the different instances. Here is an example:  
```
Host jumphost
        User ubuntu # Or cloud-user if you deployed CentOS-7 machines
        Hostname <Floating-IP>

Host behind-jh-1
        User ubuntu # Or cloud-user if you deployed CentOS-7 machines
        ProxyJump jumphost

Host behind-jh-2
        User ubuntu # Or cloud-user if you deployed CentOS-7 machines
        ProxyJump jumphost

Host behind-jh-3
        User ubuntu # Or cloud-user if you deployed CentOS-7 machines
        ProxyJump jumphost

Host behind-jh-4
        User ubuntu # Or cloud-user if you deployed CentOS-7 machines
        ProxyJump jumphost
```

You can write any name next to `Host`. Usually, we enter the same as our instances deployed.


## Cleanup
To clean everything, run this command:  
```sh
ansible-playbook -e network=project_xxxx -e state=absent main.yaml
```
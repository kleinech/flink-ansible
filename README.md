# Ansible-Flink
Ansible Playbook to automatically install flink-instances on OpenStack
## Requirements:
- Install:
    - nova
    - python >=2.7
    - shade
    - ansible 2.x


- Set OpenStack OS_* environment variables (__required__):
  1. Download OpenStack RC File (OpenStack>Compute>Access & Security>API Access)
  2. `cd flink-ansible folder`
  3. `source rc_file`
  4. (Test with `nova list`


- Create a new ssh key for ssh connetions to OpenStack instances with the name `flink`:
  1. Import public key to OpenStack (OpenStack>Compute>Access & Security>Key Pairs>Import Key Pair)
  2. Copy private key with name=`flink` to ansible-flink/certificates
  3. (the name of the private key file must match the variable `group_vars/all:ansible_ssh_private_key_file` and the name of the key name in OpenStack must match the variable `group_vars/all:private_key_name`)


- Create new Security Group with name=`flink`:
  1. OpenStack>Compute>Access & Security>Security Groups>Create Security Group
  2. Add rule for ssh access (port 22)
  3. Add rule for flink ports (8081, 8080?, 6123?)
  4. (The os_server module will automatically add the security group with name `group_vars/all:sec_group`: `flink` to the instances!)

## Start Playbook
- Start Playbook (Debug): `ansible-playbook -i hosts site.yml -K -vvv`
- Start Playbook: `ansible-playbook -i hosts site.yml -K`

## What does the playbook do?
1. Creates new instances on the OpenStack server with the os_server module
2. Saves the FloatingIPs in the host file
3. Installs flink and all necessary dependencies on the instances
4. (TODO) Setup and start the flink cluster
5. (The content of the hosts file will be overwritten in every playbook call!)

# [Ansible](http://docs.ansible.com/ansible/) Playbook Tutoiral
## Ansible Playbook Structure
1. Target Section: defines on which hosts or host the playbook will be executed.
        # Defined in hosts file
        hosts: setuphost

2. Variable Section: define variables for the playbook.
        # Define vars
        vars:
            var1: 'Test variable'
3. Task Section: ordered list of module the playbook will run.
        #Define tasks
        tasks:
            - name: Test copy
              copy: scr=fileA dest=fileB
            - name: Install pkg
              apt: name=git state=present

## Ansible Roles
[Roles](http://docs.ansible.com/ansible/playbooks_roles.html#roles) are a way to automatic load certain files:
- files
- templates
- tasks
- handlers
- vars
- defaults
- meta

Special syntax (start a role with parameters
```
roles:  
  -  { role: ROLENAME, variable1: VAR1, variable2: VAR2 }
```

## Ansible Openstack Modules
Ansible 2.0 offers a new set of [OpenStack modules](http://blog.oddbit.com/2015/10/26/ansible-20-new-openstack-modules/). Important for creating a Flink-Ansible instance is for example the [os_server](http://docs.ansible.com/ansible/os_server_module.html) module:
  ```
  os_server:
      name: "{{iname}}"
      state: present
      image: ubuntu-14.04-trusty-server-cloudimg
      flavor: m1.small
      floating_ip_pools:
        - float
      key_name: flink
      security_groups:
        - default
        - "{{sec_group}}"
      nics:
        - net-id: 8ba50f96-c076-4242-b975-1dbd83e318dd
  ```

## Ansible `group_vars`
Folder for variables (`all` file):
- Setting the private keyfile for ssh connections (__important__)
- Setting ssh arguments to solve host key checking problems
```
ansible_ssh_private_key_file: certificates/flink
ansible_ssh_common_args: -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no
```

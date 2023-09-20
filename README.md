# Ansible_test
**Testing**
with three virtual machines( Rocky9.2) in VMware17

## inventory
- hosts
- parameters
- groups


```shell

# without configuration, check servers connection
ansible all --key-file ~/.ssh/ansible -i inventory -m ping

# with ansible.cfg, check servers connection
ansible all -m ping 

# check hosts in management
ansible all --list-hosts

# run command to hosts
ansible [host_name] -m [module_name] [additions]

# example for single
ansible ubuntu@vm01 -m command "free -m"
ubuntu@vm01 | CHANGED | rc=0 >>
               total        used        free      shared  buff/cache   available
Mem:            1898         205        1141           0         551        1538

# example for all hosts
ansible all -m command "free -m"
ubuntu@vm01 | CHANGED | rc=0 >>
               total        used        free      shared  buff/cache   available
Mem:            1898         205        1141           0         551        1538
Swap:              0           0           0
ubuntu@vm02 | CHANGED | rc=0 >>
               total        used        free      shared  buff/cache   available
Mem:            1898         176        1374           0         347        1570
Swap:              0           0           0
ubuntu@vm03 | CHANGED | rc=0 >>
               total        used        free      shared  buff/cache   available
Mem:            1898         204        1148           0         545        1539
Swap:              0           0           0

# update
ansible all -m dnf -a update_cache=true

# install package
ansible all -m dnf -a name=[package_name] --become --ask-become-pass

# check the hosts info
ansible all -m setup -a ["filter=[condition]"]
```

## Playbook

example:
```shell
- hosts: [all / group_name]
  become: [true/false]
  tasks:
  
  -name: [describe this operation]
    [operation_name]:
      [option_name]: [option_value]
   when: [item_name] == [value] and [another condition]
   when: [item_name] in ["","",...]
```
- In `inventory` file, things after host are parameters, which can use in playbook.yml by `{{ [parameter] }}`. more details see `improved\_install.yml` or `improved\_uninstall.yml`. 
- `package` is a genetic package manager for most of distro package managers, like `apt`, `yum`, `dnf` and forth.
- Targeting specific nodes by groups names which setting in `inventory`.

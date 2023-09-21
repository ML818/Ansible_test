# Ansible_test
**Testing**
with three virtual machines( Rocky9.2) in VMware17

## inventory
- hosts
- parameters
- groups
```shell
[group_name]
host_name parameters...
...
```
## ansible.cfg
```
[defaults]
inventory = inventory
private_key_file = [path_to_ansible_private_key]
remote_user = [user_name]
```
- `inventory`: hosts list
- `private\_key\_file`: connect to hosts by it
- `remote\_user`: execute commands by the user defaultly

## Operations by Commands

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
   tags: [...]
    [module_name]:
      [option_name]: [option_value]
   when: [item_name] == [value] and [another condition]
   when: [item_name] in ["","",...]
```
- `become` is for sudo
- In `inventory` file, things after host are parameters, which can use in playbook.yml by `{{ [parameter] }}`. more details see `improved\_install.yml` or `improved\_uninstall.yml`. 
- `package` is a genetic package manager for most of distro package managers, like `apt`, `yum`, `dnf` and forth.
- Targeting specific nodes by groups names which setting in `inventory`.
- `tags` which always behind name of tasks
	- check more details about tags by `ansible-playbook --list-tags some.yml`.

### Modules

#### service
> change specific service status

```shell
- name: [describes]
  tags: ...
  service:
    name: [service_name]
    state: [started / restarted]
    enabled: true
  when: [conditions]
```

#### lineinfile
> modify content in specific file at specific line number

```shell
- name: ...
  tags: ...
  lineinfile:
    path: [where is the file you want to change]
    regexp: '[regular expression]'
    line: [changed content]
  when: ...
  register: [mark_name]
```
We can use the mark\_name in when statement, when it changed. Example is `service.yml`. 

#### user
> create user
```
user:
  name: [user\_name]
  groups: [group\_name]
```

#### authorized\_key
> add authorized key to specific user
```
authorized_key:
  user: [user_name]
  key: [public_key_value]
```

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
```yaml
- hosts: [all / group_name]
  become: [true/false]
  tasks:
  
  -name: [describe this operation]
   tags: [...]
    [module_name]:
      [option_name]: [option_value]
   when: [item_name] == [value] and [another condition]
   when: [item_name] in ["","",...]
   notify: [name in handler]
```
- `become` is for sudo
- In `inventory` file, things after host are parameters, which can use in playbook.yml by `{{ [parameter] }}`. more details see `improved\_install.yml` or `improved\_uninstall.yml`. 
- `package` is a genetic package manager for most of distro package managers, like `apt`, `yum`, `dnf` and forth.
- Targeting specific nodes by groups names which setting in `inventory`.
- `tags` which always behind name of tasks
	- check more details about tags by `ansible-playbook --list-tags some.yml`.
- `notify`: if something changed, it will execute.
### Modules

#### copy
> copy specific file to destination
```yaml
copy:
  src: [path to file]
  dest: [destination path]
  owner: ...
  group: ...
  mode: ...
```
#### package
> it can replace `apt` or `dnf` and so forth.
```yaml
package:
  name: [app_name]
  state: [present / absent]
  update_cache: [yes / no]
```
- `present` : install
- `absent` : uninstall
- `update_cache` : update package management repository.

#### unarchive
> terraform installation example
```yaml
unarchive:
  src: https://releases.hashicorp.com/terraform/1.5.7/terraform_1.5.7_linux_386.zip
  dest: /usr/local/bin
  remote_src: true
  mode: 0755
  owner: root
  group: root
```

#### service
> change specific service status

```yaml
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

```yaml
- name: ...
  tags: ...
  lineinfile:
    path: [where is the file you want to change]
    regexp: '[regular expression]'
    line: [changed content]
  when: ...
  register: [mark_name]
```
We can use the `mark_name` in `when` statement, when it changed. Example is `service.yml`. 

#### user
> create user
```yaml
user:
  name: [user\_name]
  groups: [group\_name]
```

#### authorized\_key
> add authorized key to specific user
```yaml
authorized_key:
  user: [user_name]
  key: [public_key_value]
```
#### template
> ssh configuration template example
```yaml
template:
  src: "{{ ssh_templates_file }}"
  dest: /etc/ssh/sshd_config
  owner: root
  group: root
  mode: 0644
notify: restart_sshd
```


### Some directories
#### files
> The path is `.` when operations commands in yaml file

#### roles
> Simplified yaml file by specific groups, write details of tasks in `[group_name]/tasks/main.yml` in `roles`. function of `files` directory is the same as former.  

`main.yml`
```yaml
- name: ...
  tags: ...
    [module_name]:
      ...
```

`playbook.yml`
```yaml
- hosts: [group_name]
  become: ...
  roles:
    - [group_name]
```

#### host\_vars
> it stores parameters for specific hosts, one host on one file which is `yaml`. Don't need put parameters in `inventory`.
> Details in `host_vars` in this repository.

#### handler
- its position is `roles/[group_name]/handlers`
- a yaml file which name is `main.yml` in `handlers`
- `main.yml`	
  ```yaml
  - name: ...
    tags: ...
      [module_name]:
        ...
  ```
- it toggles by `notify: [name]`, when something changed. Example is in `roles/web_servers/tasks/main.yml`
  - `[name]`: the name is in `main.yml` : `-name: [this_content]`

#### templates
- its position is `roles/[group_name]/templates`
- it store specific configuration template files
- it works with module `template`, and content of `src` is here
- its path is the same as `files`

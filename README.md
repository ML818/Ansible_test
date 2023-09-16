# Ansible_test
**Testing**
with three vms in multipass


```bash

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
```

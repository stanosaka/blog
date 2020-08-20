---
title: ansible
date: 2019-06-28 10:23:15
tags: [ansbile]
---

# Ansible configuration
## Installation
for ubuntu:
```
apt-get install software-properties-common
apt-add-repository ppa:ansible/ansible
apt-get update
apt-get install ansible
root@ansible-host:~# ansible --version
ansible 2.8.1
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/root/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/dist-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.12 (default, Nov 12 2018, 14:36:49) [GCC 5.4.0 20160609]
```
## Gran access to modes/machines:
- ssh-keygen
- ssh-copy-id
**Setup:**
```
cp -R /etc/ansible local
cd local
edit ansible.cfg and hosts file
```

## configuration files
```
ANSIBLE_CONFIG #an environemtn variable
ansible.cfg # in the current directory
.ansible.cfg #in the home directory
/etc/ansible/ansible.cfg
```
## Ansible inventory
`/etc/ansible/hosts`
INI format, support host variables, group variables and group of group
```
# add to /etc/ansible/hosts
[demo_hosts]
node01 ansible_user=ubuntu
node02 ansible_user=ubuntu
# add to /etc/hosts
172.31.5.185 node01
172.31.8.115 node02
```
## Verify Ansible Inventory
```
eval `ssh-agent -s`
ssh-add ansible-key.pem
ansible -m ping all
# add to /etc/ansible/hosts
[web_server]
node01 ansible_user=ubuntu

[db_server]
node02 ansible_user=ubuntu
ansible web-server -m ping
ansible db-server -m ping
```
# Local automation execution using Ansible
![architecture type](https://imgur.com/CA9XqyK)
```
Example: Ad hoc Linux echo command against a local system

#> ansible all -i "localhost," -c local -m shell -a 'echo hello DevOps World'
cat hellodevopsworld.yml
# File name: hellodevopsworld.yml
--- 
- hosts: all
 tasks:
 - shell: echo "hello DevOps world"

# Running a Playbook from the command line:
#> ansible-playbook -i 'localhost,' -c local hellodevopsworld.yml
```

**Remote automation execution using Ansible**
```
# Example command: execute the playbook hellodevopsworld.yml against rpi-01
ansible-playbook -i 'rpi-01,' -c local  ~/Learning/ansible/hellodevopsworld.yml
```

# Run and execute ansible tasks
## Ansible Command Line
Two ways: ad-hoc command and playbook
### Ansible Ad-hoc Commands
`ansible <target> -m <module name> -a arguments`

support parallelism: `<command> -f`

`ansible demo_host -m copy -a "src=/tmp/test1 dest=/tmp/test1"` #-m means module, -a optional provided a list of arguments

```
ansible-doc -l
ansible-doc copy
ansible-doc -l|grep shell
touch /tmp/test1
ansible demo_hosts -m copy -a "src=/tmp/test1 dest=/tmp/test1"
```
## Ansible Facts
A fact is a detail or piece of information collect from remote host. Can be use for grouping node, or filter node.
```
ansible all -m setup
ansible demo_host -m setup 
```

## Ansible Variables
Valid Variable names
- Cannot use "-" (hyphen)
- Can be alphanumeric
- Should start with an alphabet

Ansible Variables Naming Conventions

can define variables in Inventory, playbooks file and roles.

## Ansible Sections
- Target Section: sepecify the host group
- Variable Section
- Task Section : list all the task
- Handler Section
- Loops
- Conditionals
- Until
- Notify
## Ansible playbooks
```
root@ansible-host:/home/ubuntu# cat sample.yaml
---
- hosts: demo_hosts
  vars:
      package1 : "nginx"
      package2 : "wget"
  tasks:
   - name: Installing package nginx
     apt: pkg=nginx state=installed update_cache=true
     become: true
   - name: Installing package wget
     apt: name={{ package2 }} state=installed update_cache=true
     become: true
   - name: Copying test1 file
     copy: src=/tmp/test11 dest=/tmp/test11

ansible-doc apt ##check the manual for ansible apt
ansible-playbook sample.yaml ## run ansible playbook
```
# Deep Dive Into Ansible Playbooks

## Install Apache with Ansible Playbook
```
root@ansible-host:~# cat apache_install.yaml
---
- hosts: web_portal
  tasks:
    - name: Apt get update
      apt: update_cache=yes

    - name: Install Apache2
      apt: name=apache2 update_cache=no

    - name: Copy data files
      copy: src=index.html dest=/var/www/html/

    - name: Stop the service
      service: name=apache2 state=stopped
root@ansible-host:~# ansible-playbook apache_install.yaml -b
```
## Run and Stop Service

## Template
```
root@ansible-host:~# cat current.html.j2
this is my current file -
my hostname is - {{ ansible_hostname }}
root@ansible-host:~# cat apache_install.yaml
---
- hosts: web_portal
  tasks:
    - name: Apt get update
      apt: update_cache=yes

    - name: Install Apache2
      apt: name=apache2 update_cache=no

    - name: Copy data files
      copy: src=index.html dest=/var/www/html/

    - name: Stop the service
      service: name=apache2 state=stopped

    - name: Copy template file
      template: src=current.html.j2 dest=/var/www/html/current.html
      notify:
        - Start apache
  handlers:
    - name: Start apache
      service: name=apache2 state=restarted
### dry run ansible
ansible-playbook apache_install.yaml -b --check
```
## Debug Statements In playbook
```
root@ansible-host:~# cat apache_install.yaml
---
- hosts: web_portal
  tasks:
    - name: Apt get update
      apt: update_cache=yes

    - name: Install Apache2
      apt: name=apache2 update_cache=no

    - name: Copy data files
      copy: src=index.html dest=/var/www/html/
      register: copy_status

    - name: Stop the service
      service: name=apache2 state=stopped

    - name: Copy template file
      template: src=current.html.j2 dest=/var/www/html/current.html
      notify:
        - Start apache

    - name: Copy status
      debug: var=copy_status

  handlers:
    - name: Start apache
      service: name=apache2 state=restarted
root@ansible-host:~# ansible-playbook apache_install.yaml -b

PLAY [web_portal] *************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************
ok: [node01]

TASK [Apt get update] *********************************************************************************************************************************
 [WARNING]: Could not find aptitude. Using apt-get instead

changed: [node01]

TASK [Install Apache2] ********************************************************************************************************************************
ok: [node01]

TASK [Copy data files] ********************************************************************************************************************************
ok: [node01]

TASK [Stop the service] *******************************************************************************************************************************
ok: [node01]

TASK [Copy template file] *****************************************************************************************************************************
ok: [node01]

TASK [Copy status] ************************************************************************************************************************************
ok: [node01] => {
    "copy_status": {
        "changed": false,
        "checksum": "9be1fe2ae3529c22cf8a1b0fd1edc53048dcf277",
        "dest": "/var/www/html/index.html",
        "diff": {
            "after": {
                "path": "/var/www/html/index.html"
            },
            "before": {
                "path": "/var/www/html/index.html"
            }
        },
        "failed": false,
        "gid": 0,
        "group": "root",
        "mode": "0644",
        "owner": "root",
        "path": "/var/www/html/index.html",
        "size": 24,
        "state": "file",
        "uid": 0
    }
}

PLAY RECAP ********************************************************************************************************************************************
node01                     : ok=7    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

## Loops, Conditionals and until section
```
root@ansible-host:~# cat apache_install.yaml
---
- hosts: web_portal
  tasks:
    - name: Apt get update
      apt: update_cache=yes

    - name: Install Apache2, nginx, nmap
      apt: name={{ item }} update_cache=no
      with_items:
         - apache2
         - nginx
         - nmap

    - name: Copy data files
      copy: src=index.html dest=/var/www/html/
      register: copy_status

    - name: Stop the service
      service: name=apache2 state=stopped

    - name: Copy template file
      template: src=current.html.j2 dest=/var/www/html/current.html
      notify:
        - Start apache

    - name: Copy status
      debug: var=copy_status

  handlers:
    - name: Start apache
      service: name=apache2 state=restarted
```
# Putting it all together with ansible
## Ansible vault
ansible-vault create foo.yml
ansible-vaule view foo.yml
ansible-playbook site.yml -ask-vault-pass

## Common Ansible Modules
- Setup module
ansible node01 -m setup
- File module
ansible-doc file
- Yum module
for redhat, centos
- Apt module
for debian, ubuntu
- Service module
for start and stop service
- Copy module
which is used for copy the file
- User module
ansible-doc user
ansible node01 -m user -a "user=stancloud" -b
- Command module
ansible-doc command
- Shell module
ansible node01 -m shell -a "tail /var/log/syslog"
## Ansible Roles
Roles are similar to project.
### roles directory structure
```
ansible-galaxy init stan
- stan was created successfully
root@ansible-host:/tmp# tree stan
stan
├── defaults
│   └── main.yml
├── files
├── handlers
│   └── main.yml
├── meta
│   └── main.yml
├── README.md
├── tasks
│   └── main.yml
├── templates
├── tests
│   ├── inventory
│   └── test.yml
└── vars
    └── main.yml
roles/
  role-name/
     files/
     templates/ #j2 files
     tasks/
        main.yml
     handlers/
        main.yml
     vars/
        main.yml
     defaults/
        main.yml
     meta
```
- Common role
- mySQL role
- Apache role
- WordPress role
### Deploy wordpress

## Ansible Galaxy
root@ansible-host:/tmp# ansible-galaxy install bennojoy.nginx
- downloading role 'nginx', owned by bennojoy
- downloading role from https://github.com/bennojoy/nginx/archive/master.tar.gz
- extracting bennojoy.nginx to /root/.ansible/roles/bennojoy.nginx
- bennojoy.nginx (master) was installed successfully
[jinja](http://jinja.pocoo.org)
# Manage aws cloud resouces with ansible
## manage aws ec2 instances with ansible
ansible-doc -l|grep aws
ansible-doc -l|grep ec2
## Deploy new aws ec2 instances using ansible playbook
```
cat ec2_create.yaml
---
- hosts: localhost
  tasks:
    - name: Create AWS instances
      ec2:
        key_name: ansible-key
        instance_type: t2.micro
        image: ami-001dae151248753a2
        count: 3
        vpc_subnet_id: subnet-20615347
        assign_public_ip: no
        region: ap-southeast-2

ansible-playbook -i "localhost," -c local ec2_create.yaml
```

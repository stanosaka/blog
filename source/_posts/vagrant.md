---
title: vagrant
date: 2019-08-03 23:35:53
tags:
  - vagrant
  - Hashicorp
---
Vagrant can be used for automatically provisioning VMs, and even whole development environments.

On ubuntu host:
```
$ sudo apt install vagrant
$ vagrant --version
$ mkdir Vagrant;cd Vagrant
$ vagrant init
# Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "base"
$ sed -i 's#config.vm.box = "base"#config.vm.box = "centos/7"#g' Vagrantfile
$ vagrant up
$ vagrant ssh
$ vagrant destroy
```
**Components**
- Providers:
   - Backend of Vagrant
   - VirtualBox
   - VMware
   - Hyper-V
   - vCloud
   - AWS
- Boxes:
   - Predefined images
   - [Public Vagrant box catalog](https://app.vagrantup.com/boxes/search)
- Vagrantfile:
   - A Ruby file
   - How many VMs
   - Configure VMs
   - Provision VMs
   - Committed to version control
![example of Vagrantfile](https://i.imgur.com/lb7muP9.png)
- Provisioners:
   - Automatically install software, alter configurations
   - Boxes may not be a complete use case for you
   - Multiple options
   - Shell Script
   - Ansible
   - Chef
   - Docker
   - Puppet
![workflow of vagrant](https://i.imgur.com/U5cISEK.png)

**Operations**
- Adding a vagrant box:
   - Syntax: vagrant box add <name> <url> <provider>
   - Example: vagrant box add ubuntu/trusty32
- Listing and removing vagrant boxes:
   - vagrant box list
   - vagrant box remove
- Creating a VM environment:
   - Syntax: vagrant init <your box name>
   - Example: vagrant init ubuntu/trusty32
- Starting a VM environment:
   - vagrant up ubuntu/trusty32
   - vagrant up 
- Connecting:
   - vagrant ssh ubuntu/trusty32
   - vagrant ssh
- Stopping, restarting, and destroying
   - vagrant halt
   - vagrant reload
   - vagrant destroy

**Provisioning of vagrant**
- Creating a VM and provisioning it with Apache installed and prot forwarding
- Add theses lines in Vagrantfile:
```
  config.vm.network "forwarded_port", guest: 80, host: 8080
  config.vm.provision 'shell', path: 'provision.sh'
```
- Create provision.sh file with these entries:
  - sudo apt-get update
  - sudo apt-get install -y apache2

- Destroy and start the virtual machine:
  - vagrant destroy
  - vagrant up

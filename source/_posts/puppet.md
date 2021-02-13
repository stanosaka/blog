---
title: puppet
date: 2020-04-26 22:41:23
tags: puppet
---

# Puppet Code Directory Layout #

environments/production/:
manifests/:Puppet manifests
modules/: Puppet modules
data/: Hiera data
hiera.yaml: Hiera configuration

# add cert autosign

cat /etc/puppetlabs/puppet/autosign.conf
add the hostname or *.foobar

add a line in /etc/puppetlabs/puppet/puppet.conf
autosign = true

# Puppet 5 essentials
## resources, parameters, and properties

Resources are the elementray builing blocks of manifests.
```
# cat puppet_service.pp
service { 'puppet':
  ensure => 'stopped',
  enable => false,
} 
```
Each has a type (service) and a name or title (puppet). Each resource is unique to a manifest, and can be referenced by the combination of its
type and name, such as Server["puppet"].
A resource comprises a list of zero or more attributes. 
An attribute is a key-value pair, such as `enable => fales`.

Puppet differentiates between two different attributes: parameters and properties.
Parameters describe the way that Puppet should deal with a resource type.
Properties describe a specific setting of a resource.
Certain parameters are available for all resource types (metaparameters), and some names are just very common, such as ensure. The service type supports the ensure property, which represents the status of the managed process. Its enabled property, on the other hand, relates to the system boot configuration (with respect to the service in question).

```
cat puppet_service_provider.pp 
service { 'puppet':
  ensure   => 'stopped',
  enable   => false,
  provider => 'upstart',
}
```
The provider parameter tells Puppet that it need to interact with the upstart subsystem to control its background service.

The difference between parameters and properties is that the parameter merely indicates how Puppet should manage the resource, not what a desired state is.
Puppet will only take action on property values. In this example, these are ensure => 'stopped' and enable => false. For each such property, Puppet will perform the following tasks:

- Test whether the resources is already in sync with the target state
- If the resource is not in sync, it will trigger a sync action

Properties can be out of sync, whereas parameters cannot.

## Dry testing your manifest
`puppet apply puppet_service.pp --noop`

## Using variables
Any variable name is always prefixed with the $ sign:
```
$download_server = 'img2.example.net'
$url = "https://${download_server}/pkg/example_source.tar.gz" 
```
## Variable types
Four variable types: strings, arrays, hashed, and Booleans
```
$a_bool = true
$a_string = 'This is a string value'
$an_array = [ 'This', 'forms', 'an', 'array' ]
$a_hash = {   
   'subject'   => 'Hashes',  
   'predicate' => 'are written',  
   'object'    => 'like this',  
   'note'      => 'not actual grammar!',  
   'also note' => [ 'nesting is',
   { 'allowed'   => ' of course' } ], 
}
```
## Data types
Puppet has core data types and abstract data types. The core data types are the most commonly used types of data, such as string or integer, whereas abstract data types allow for more sophisticated type validation, such as optional or variant.

## Adding control structures in manifests

if/else block:
```
if 'mail_lda' in $needed_services {  
   service { 'dovecot': enable => true }
} else {  
   service { 'dovecot': enable => false }
}
```
case statement:
```

```
# How to add a new module in puppet
## 1. modified Puppetfile
```
mod 'puppetlabs-lvm',  '1.4.0'
```

## 2. ssh to puppet master
```
cd /etc/puppetlabs/code/environments/production/
librarian-puppeet install --verbose
```

## 3. modified data/common.yaml and base.pp
```
profile::base::enable_lvm: false
vim base.pp
if $enable_lvm {
   include lvm
}

```
## 4. new file data/node-overrides/hostname.yaml
```
profile::base::enable_lvm: true
lvm::volume_groups:
  mysql_vg:
    createonly: true
    physical_volumes:
      /dev/sdb:
    logical_volumes:
      ipscape_db:
        size: undef 
        mountpath: /dev/mysql_vg/ipscape_db
        mountpath_require: true

```
## 5. file_line 
[file_line types](https://forge.puppet.com/modules/puppetlabs/stdlib/4.9.1/readme)
```
  file_line { 'asterisk_setting':
    path => '/etc/ipscape-api/server.settings',
    line => 'asterisk_pool_max_idle = 0',
    ensure => present
  }

```

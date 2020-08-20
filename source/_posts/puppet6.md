---
title: puppet6
date: 2020-06-28 21:26:20
tags: puppet6, puppet
---
# puppet 6
## Local environment installation
### Puppet 6 download
<http://apt.puppet.com/>
<http://yum.puppet.com/puppet6/>
<http://downloads.puppet.com/windows/puppet6/>
<http://yum.puppet.com/mac/puppet6/>
```
vagrant plugin install vagrant-hostmanager
vagrant plugin install vagrant-vbguest
vagrant --version
vboxmanage --version
git --version
vagrant status
vagrant up centos
vagrant hostmanager # updating hosts file on the vm 

```
## Types and providers
```
[root@centos ~]# puppet resource package
package { 'NetworkManager':
  ensure   => '1:1.18.4-3.el7',
  provider => 'yum',
}

[root@centos ~]# puppet resource package|less
[root@centos ~]# gem list

*** LOCAL GEMS ***

bigdecimal (1.2.0)
io-console (0.4.2)
json (1.7.7)
psych (2.0.0)
rdoc (4.0.0)
[root@centos ~]# puppet resource package bigdecimal
package { 'bigdecimal':
  ensure   => ['1.2.0'],
  provider => 'gem',
}
[root@centos ~]# puppet resource package git
package { 'git':
  ensure   => 'purged',
  provider => 'yum',
}
```

## Puppet Master installation

[Hands-on-puppet-6](https://github.com/PacktPublishing/Hands-On-Infrastructure-Automation-with-Puppet-6-V- "puppet 6")

## Puppet communication

### on centos:
```
[vagrant@centos ~]$ curl -v https://puppet:8140/puppet-ca/v1/certificate/ca

[vagrant@centos ~]$ curl -v https://puppet:8140/puppet-ca/v1/certificate/ca -k
* About to connect() to puppet port 8140 (#0)
*   Trying 192.168.50.100...
* Connected to puppet (192.168.50.100) port 8140 (#0)
* Initializing NSS with certpath: sql:/etc/pki/nssdb
* skipping SSL peer certificate verification
* NSS: client certificate not found (nickname not specified)
```

### on puppet:
```
root@puppet ~]# puppetserver ca list
Requested Certificates:
    centos.example.com       (SHA256)  42:79:9E:79:A7:17:92:88:4C:E9:DE:A2:75:F0:03:5E:6A:7C:C4:D0:6B:AF:0E:F4:69:38:B8:9D:0F:E5:AE:5E
[root@puppet ~]# puppetserver ca sign --cert centos.example.com
Successfully signed certificate request for centos.example.com
```
### check cert expired date
```
[root@centos ~]# cd /etc/puppetlabs/puppet/ssl/
[root@centos ssl]# find .
.
./certs
./certs/ca.pem
./certs/centos.example.com.pem
./public_keys
./certificate_requests
./private_keys
./private_keys/centos.example.com.pem
./private
./crl.pem
root@centos ssl]# openssl x509 -in certs/ca.pem -text -noout|less
[root@centos ssl]# openssl x509 -in certs/centos.example.com.pem -text -noout|less
[root@centos ssl]# openssl verify -CAfile certs/ca.pem certs/centos.example.com.pem
certs/centos.example.com.pem: OK

[root@centos ssl]# cat certs/ca.pem crl.pem > ca_crl.pem
[root@centos ssl]# openssl verify -CAfile ca_crl.pem -crl_check certs/centos.example.com.pem
certs/centos.example.com.pem: OK
[root@centos ssl]# openssl x509 -in certs/centos.example.com.pem -modulus -noout
Modulus=C57D960784B36EB5E316C1CFA95424062207356C0ACA2271C060120E77EDE0D44347D109982ABDC46F122AFAF85EE83CBE9684AA9E2164D40246D2C72CE9BB67C9C6544282829CC6B1D9100B0A32BAD630DB65EFA338ECE9B941CE2C19003A2AF8BAE7DDADA2A4F7279CC0E2916AB37A058F9CF8316D35BFA45BF5B0E178ACE928F77846FE0DB035CB584A7C2622DE7A48C959814E1AFB9F6B5F7F0BAC284F0EC336D1C376691F388328B1AF0037D1789C8D1D5D12AD72014706594B9EF9463B10EF3949BD193201334B46724525D164B57F3A1BB350C2915D9476FB66547B5F4E861FC60A4309D8739910E4603C3E5BF536D3BB4581F0789044110AF8350AB1E4189459035EDD1CD13B737D1A17F96026B83686EED285A51F06C5B9E840A6FCDC29C13ACBE7E8787EDC293479D0CFCDE95610C5E93A377F8C1F962001EF354F39359E39855CCC9FF1C9E7548FBA6143784F80C820D5EFE59F96AC78196D560195260F3E1B6A8A0354EDF3F5D368EC4EAF0F3C91CBF88E436C529136ACCA1A9E6CA7ABDCF22763C38A6E6B9CCF75C9570B03D74BB54A50D114B256A540AF6973A1327D9C4CAC7C244BCA33AE27B96668F1CC5ED8D0B808E150ECC56F2489F0CECEB18D8A6E403DF434246D30DE327FF53D9B770C80194ABE124E84E15737B57543E92D50B146BE5BB56618E42783E80E8D5FE18B1E8AA509FCE64FFDA3521019
[root@centos ssl]# openssl x509 -in certs/centos.example.com.pem -modulus -noout|sha256sum
1527c07dedde5aed5061b7b2416c6d942c7cb66cd457ee545b02ef9934ca2e45  -
[root@centos ssl]# openssl rsa -in private_keys/centos.example.com.pem -modulus -noout
Modulus=C57D960784B36EB5E316C1CFA95424062207356C0ACA2271C060120E77EDE0D44347D109982ABDC46F122AFAF85EE83CBE9684AA9E2164D40246D2C72CE9BB67C9C6544282829CC6B1D9100B0A32BAD630DB65EFA338ECE9B941CE2C19003A2AF8BAE7DDADA2A4F7279CC0E2916AB37A058F9CF8316D35BFA45BF5B0E178ACE928F77846FE0DB035CB584A7C2622DE7A48C959814E1AFB9F6B5F7F0BAC284F0EC336D1C376691F388328B1AF0037D1789C8D1D5D12AD72014706594B9EF9463B10EF3949BD193201334B46724525D164B57F3A1BB350C2915D9476FB66547B5F4E861FC60A4309D8739910E4603C3E5BF536D3BB4581F0789044110AF8350AB1E4189459035EDD1CD13B737D1A17F96026B83686EED285A51F06C5B9E840A6FCDC29C13ACBE7E8787EDC293479D0CFCDE95610C5E93A377F8C1F962001EF354F39359E39855CCC9FF1C9E7548FBA6143784F80C820D5EFE59F96AC78196D560195260F3E1B6A8A0354EDF3F5D368EC4EAF0F3C91CBF88E436C529136ACCA1A9E6CA7ABDCF22763C38A6E6B9CCF75C9570B03D74BB54A50D114B256A540AF6973A1327D9C4CAC7C244BCA33AE27B96668F1CC5ED8D0B808E150ECC56F2489F0CECEB18D8A6E403DF434246D30DE327FF53D9B770C80194ABE124E84E15737B57543E92D50B146BE5BB56618E42783E80E8D5FE18B1E8AA509FCE64FFDA3521019
[root@centos ssl]# openssl rsa -in private_keys/centos.example.com.pem -modulus -noout|sha256sum
1527c07dedde5aed5061b7b2416c6d942c7cb66cd457ee545b02ef9934ca2e45  -
```
## Catalogs and environments
When apply puppet code to a machine or node, need to compile the code into a catalog.
The catalog is a representation of the code as a graph.

```
[root@puppet manifests]# cd /opt/puppetlabs/puppet/cache/
[root@puppet cache]# ls
clientbucket  client_data  client_yaml  facts.d  lib  locales  preview  reports  state
[root@puppet cache]# cd client_data/catalog/
[root@puppet catalog]# cat puppet.example.com.json |python -m json.tool|less
```
### Environments
```
[root@puppet environments]# ls
production  sandbox
[root@puppet environments]# cd sandbox/
[root@puppet sandbox]# ls
data  environment.conf  hiera.yaml  manifests  modules
[root@puppet sandbox]# cd manifests/
[root@puppet manifests]# ls
site.pp
[root@puppet manifests]# vi site.pp
[root@puppet manifests]# puppet agent -t --environment sandbox
Info: Using configured environment 'sandbox'
Info: Retrieving pluginfacts
Info: Retrieving plugin
Info: Retrieving locales
Info: Caching catalog for puppet.example.com
Info: Applying configuration version '1593430390'
Notice: This is the sandbox environment
Notice: /Stage[main]/Main/Node[default]/Notify[This is the sandbox environment]/message: defined 'message' as 'This is the sandbox environment'
Notice: Applied catalog in 0.01 seconds
[root@puppet manifests]# puppet agent -t
Info: Using configured environment 'production'
Info: Retrieving pluginfacts
Info: Retrieving plugin
Info: Retrieving locales
Info: Caching catalog for puppet.example.com
Info: Applying configuration version '1593430423'
Notice: This is production, only approved code is permitted
Notice: /Stage[main]/Main/Node[default]/Notify[This is production, only approved code is permitted]/message: defined 'message' as 'This is production, only approved code is permitted'
Notice: Applied catalog in 0.01 seconds
[root@puppet manifests]# vi /etc/puppetlabs/puppet/puppet.conf
add:
[agent]
environment=sandbox
[root@puppet manifests]# puppet agent -t
Info: Using configured environment 'sandbox'
Info: Retrieving pluginfacts
Info: Retrieving plugin
Info: Retrieving locales
Info: Caching catalog for puppet.example.com
Info: Applying configuration version '1593430508'
Notice: This is the sandbox environment
Notice: /Stage[main]/Main/Node[default]/Notify[This is the sandbox environment]/message: defined 'message' as 'This is the sandbox environment'
Notice: Applied catalog in 0.01 seconds
```
![environments](https://i.imgur.com/BGh6QOP.png)

## Modules and Puppet Forge
```
[root@puppet ~]# puppet module install puppetlabs-apache
Notice: Preparing to install into /etc/puppetlabs/code/environments/production/modules ...
Notice: Downloading from https://forgeapi.puppet.com ...
Notice: Installing -- do not interrupt ...
/etc/puppetlabs/code/environments/production/modules
└─┬ puppetlabs-apache (v5.4.0)
  ├─┬ puppetlabs-concat (v6.2.0)
  │ └── puppetlabs-translate (v2.2.0)
  └── puppetlabs-stdlib (v6.3.0)
[root@puppet ~]# cat /etc/puppetlabs/puppet/puppet.conf
# This file can be used to override the default puppet settings.
# See the following links for more details on what settings are available:
# - https://puppet.com/docs/puppet/latest/config_important_settings.html
# - https://puppet.com/docs/puppet/latest/config_about_settings.html
# - https://puppet.com/docs/puppet/latest/config_file_main.html
# - https://puppet.com/docs/puppet/latest/configuration.html
[master]
vardir = /opt/puppetlabs/server/data/puppetserver
logdir = /var/log/puppetlabs/puppetserver
rundir = /var/run/puppetlabs/puppetserver
pidfile = /var/run/puppetlabs/puppetserver/puppetserver.pid
codedir = /etc/puppetlabs/code
[agent]
environment=sandbox
[root@puppet ~]# puppet config print modulepath
/etc/puppetlabs/code/environments/production/modules:/etc/puppetlabs/code/modules:/opt/puppetlabs/puppet/modules
[root@puppet ~]# puppet config print environment
production
[root@puppet ~]# puppet config print environment --section=agent
sandbox
[root@puppet production]# which gem
/bin/gem
[root@puppet production]# vi Puppetfile
mod 'puppetlabs/vcsrepo'
mod 'puppetlabs/apache'
mod 'first',
   :git => 'https://github.com/uphillian/uphillian-sample.git',
   :ref => 'master'
   
[root@puppet production]# export PATH=/opt/puppetlabs/puppet/bin:/opt/puppetlabs/bin:$PATH
[root@puppet production]# which gem
/opt/puppetlabs/puppet/bin/gem
[root@puppet production]# gem install r10k
[root@puppet production]# r10k
NAME
    r10k - Killer robot powered Puppet environment deployment
    
[root@puppet production]# r10k puppetfile install
[root@puppet production]# ls modules
apache  first  vcsrepo
```
### concat Module
[concat](https://github.com/puppetlabs/puppetlabs-concat)
[concat forge](https://forge.puppet.com/puppetlabs/concat)

```
[root@puppet production]# pwd
/etc/puppetlabs/code/environments/production
[root@puppet production]# cat Puppetfile
mod 'puppetlabs/vcsrepo'
mod 'puppetlabs/apache'
mod 'puppetlabs/stdlib'
mod 'puppetlabs/concat'
mod 'first',
   :git => 'https://github.com/uphillian/uphillian-sample.git',
   :ref => 'master'
[root@puppet production]# r10k puppetfile install
[root@puppet production]# cd modules/
[root@puppet modules]# ls
apache  concat  first  stdlib  vcsrepo
[root@puppet modules]# cd ../manifests/
[root@puppet manifests]# ls
site-2.pp  site.pp
[root@puppet manifests]# cat site-2.pp
concat {'/etc/motd': }

package { 'epel-release':
  ensure => 'installed',
}

package { 'figlet':
  ensure => 'installed',
  require => Package['epel-release'],
}

exec {'motd.hostname':
  path    => '/bin:/usr/bin',
  command => "figlet $hostname >/etc/motd.hostname",
  creates => '/etc/motd.hostname',
  require => Package['figlet'],
}

exec {'motd.warning':
  path    => '/bin:/usr/bin',
  command => "figlet '* WARNING *'>/etc/motd.warning",
  creates => '/etc/motd.warning',
  require => Package['figlet'],
}

concat::fragment { 'hostname':
  target  => '/etc/motd',
  source  => '/etc/motd.hostname',
  order   => '01',
  require => Exec['motd.hostname'],
}

concat::fragment { 'info':
  target => '/etc/motd',
  content => "${fact('os.name')} ${fact('os.release.major')}\n",
  order => '05',
}

$disclaimer = @(END)
  -----------------------------------------------------------
  This system is for the use of authorized users only.
  Individuals using this computer system without authority,
  or in excess of their authority, are subject to having all
  of their activities on the system monitored and recorded.
  This information will be shared with law enforcement should
  any wrong doing be suspected.
  -----------------------------------------------------------

  | END

concat::fragment { 'warning':
  target => '/etc/motd',
  source => '/etc/motd.warning',
  order  => '10',
}
concat::fragment { 'disclaimer':
  target  => '/etc/motd',
  content => $disclaimer,
  order   => '20',
}

# run
puppet agent -t
[root@puppet manifests]# cat /etc/motd
                              _
 _ __  _   _ _ __  _ __   ___| |_
| '_ \| | | | '_ \| '_ \ / _ \ __|
| |_) | |_| | |_) | |_) |  __/ |_
| .__/ \__,_| .__/| .__/ \___|\__|
|_|         |_|   |_|
CentOS 7
       __        ___    ____  _   _ ___ _   _  ____
__/\__ \ \      / / \  |  _ \| \ | |_ _| \ | |/ ___| __/\__
\    /  \ \ /\ / / _ \ | |_) |  \| || ||  \| | |  _  \    /
/_  _\   \ V  V / ___ \|  _ <| |\  || || |\  | |_| | /_  _\
  \/      \_/\_/_/   \_\_| \_\_| \_|___|_| \_|\____|   \/

-----------------------------------------------------------
This system is for the use of authorized users only.
Individuals using this computer system without authority,
or in excess of their authority, are subject to having all
of their activities on the system monitored and recorded.
This information will be shared with law enforcement should
any wrong doing be suspected.
-----------------------------------------------------------
```
## Puppet Lookup
```
puppet lookup role::cms::docroot
puppet lookup role::cms::docroot --explain
puppet lookup --node ubuntu.expale.com role::cms::docroot
[root@puppet manifests]# cat scope.pp
class role::scope (
  String $example = "A scoped variable"
) {
  # do nothing
  }
[root@puppet manifests]# pwd
/etc/puppetlabs/code/environments/production/modules/role/manifests
[root@puppet manifests]# cat scope.pp
class role::scope (
  String $example = "A scoped variable"
) {
  # do nothing
  }
[root@puppet manifests]# cat /root/scope.pp
include role::scope
notify { "role::scope::example is ${::role::scope::example}": }
[root@puppet manifests]# puppet apply /root/scope.pp
Notice: Compiled catalog for puppet.example.com in environment production in 0.01 seconds
Notice: role::scope::example is A scoped variable
Notice: /Stage[main]/Main/Notify[role::scope::example is A scoped variable]/message: defined 'message' as 'role::scope::example is A scoped variable'
Notice: Applied catalog in 0.01 seconds
```

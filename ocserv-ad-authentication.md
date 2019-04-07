# How to setup ocserv for Microsoft AD Authentication

Author: Dionis Pelivan


### Scope

This Recipe provides step by step instructions on how to install, configure,
and test Microsoft AD Authentication for Openconnect Server. This recipe focuses on
generic installation instructions, from packages available on Openconnect server.
No precompiled binary packages will be used, therefore this recipe was tested 
on CentOS 7 only.

### Platforms used for testing

This Recipe was tested on the following platforms: 
  
* CentOS 7 on amd64 architecture

### Assumptions

This recipe assumes the reader has a basic understanding of a linux
system and all commands are run from a privileged user. It is recommended
to login the system using root.If not possible, execute ```su root``` or
```sudo su``` to get highest privileges.

### Prerequisite
In order to take advantage of this setup, you should join your linux server into AD domain.
The packages below are required in order to join linux to AD Domain, create home dir and so on.
``` 
 [root@vpn ~]# yum install pam-devel oddjob oddjob-mkhomedir sssd samba-common-tools realmd polkit.i686 iptables-services pam cracklib
``` 
#### join in Active Directory domain
``` 
 [root@vpn ~]# realm join YOURDOMAIN.COM --user Administrator
``` 
Open /etc/sssd/sssd.conf with text editor and replace with the below config (replace yourdomain.com acordingly)
```  
 [root@vpn ~]# vim /etc/sssd/sssd.conf
```
```
[sssd]
debug_level = 5
domains = yourdomain.com
config_file_version = 2
services = nss, sudo, pam, ssh

[domain/yourdomain.com]
debug_level = 5
ad_domain = yourdomain.com
krb5_realm = YOURDOMAIN.COM
realmd_tags = manages-system joined-with-samba
cache_credentials = True
id_provider = ad
krb5_store_password_if_offline = True
default_shell = /bin/bash
ldap_id_mapping = True
use_fully_qualified_names = False
fallback_homedir = /home/yourdomain/%u
access_provider = ad
```

### Details on lab used on this recipe
 
 * network 192.168.255.0/24 (netmask 255.255.255.0)
 * ocserv ip 192.168.255.254
 * ocserv hostname vpn

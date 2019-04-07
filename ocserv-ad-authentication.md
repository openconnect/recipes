# How to setup ocserv for Microsoft AD Authentication

Author: Dionis Pelivan


### Scope

This Recipe provides step by step instructions on how to install, configure,
and test Microsoft AD Authentication for Openconnect Server. This recipe was
tested on CentOS 7 with the EPEL packages of ocserv.

### Platforms used for testing

This Recipe was tested on the following platforms: 
  
* CentOS 7 on amd64 architecture

### Assumptions

This recipe assumes the reader has a basic understanding of a linux
system and all commands are run from a privileged user. It is recommended
to login the system using root.

### Prerequisite
In order to take advantage of this setup, you should join your linux server into AD domain.
The packages below are required in order to join linux to AD Domain, create home dir and so on.
``` 
 [root@vpn ~]# yum install oddjob oddjob-mkhomedir sssd samba-common-tools realmd polkit.i686 iptables-services pam cracklib
``` 

Additionally you need to add PAM as the authentication backend of ocserv.
The rest of this text assumes that a working PAM configuration is in place and
pam_sss is enabled.

In that case, the following lines should be present in ocserv.conf.

```
auth = "pam"
```

### Join in Active Directory domain
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


## Ocserv Installation - CentOS, RHEL, Fedora

Author: Mauro Gaspari  


### Scope
This recipe provides step by step instructions on how to install ocserv from EPEL repository on
RHEL and CentOS operating systems.

### Platforms used for testing
This Recipe was tested on the following platforms:   

- CentOS 7 on amd64 architecture
- Fedora 25 on amd64 architecture


### Assumptions

This recipe assumes the reader has a basic understanding of a GNU/linux system and all commands are
run from a privileged user. It is recommended to login the system using root. If not possible,
execute "su root" or "sudo -s" to get highest privileges.


### Requirements

- During this recipe, CentOS and RHEL users will be asked to add the EPEL repository on Centos and RHEL. ocserv and radcli packages are only available on EPEL repository.
- Fedora users do not need to add EPEL repository. ocserv and radcli packages are available on Fedora repositories.  


### Installation - CentOS and RHEL

1. Add EPEL repository to CentOS, or [see instructions for RHEL](https://fedoraproject.org/wiki/EPEL)

	```yum install epel-release```
	
2. Confirm the repository when asked. 

3. Check if the EPEL repository is enabled 

	```yum repolist enabled	```

4. Check if ocserv is available for install:  

	```yum info ocserv```

5. Install ocserv. This will install ocserv and its dependencies, including radcli radius libraries.

	```yum install ocserv```

### Installation - Fedora


1. Check if ocserv is available for install:  

	```dnf info ocserv```

1. Install ocserv. This will install ocserv and its dependencies, including radcli radius libraries.

	```dnf install ocserv```


### Final notes

This concludes **Ocserv Installation - CentOS, RHEL, Fedora** recipe. At this point Openconnect server should be ready to be configured.  


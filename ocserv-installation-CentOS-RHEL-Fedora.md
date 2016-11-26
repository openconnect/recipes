## Ocserv Installation - CentOS, RHEL, Fedora

Author: Mauro Gaspari  


###Scope
This recipe provides step by step instructions on how to install ocserv from EPEL repository on CentOS and RHEL operative systems.

### Platforms used for testing
This Recipe was tested on the following platforms:   

- CentOS 7 on amd64 architecture
- Fedora 25 on amd64 architecture


### Assumptions

- This recipe assumes the reader has a basic understanding of a GNU/linux system and all commands are run from a privileged user. It is recommended to login the system using root.If not possible, execute "su root" or "sudo su" to get highest privileges.


###Requirements
- During this recipe, CentOS and RHEL users will be asked to add the EPEL repository on Centos and RHEL. ocserv and radcli packages are available on EPEL repository.  
- Fedora users do not need to add EPEL repository. ocserv and radcli packages are available on Fedora default repositories.  



### Details on lab used on this recipe
- network 192.169.5.0/24 (netmask 255.255.255.0)
- ocserv ip 192.168.5.254
- ocserv hostname fw01


### Installation - CentOS and RHEL

1. Add EPEL repository to CentOS or RHEL 

	```
	yum install epel-release
	
	```

2. Confirm the repository when asked. 

3. Check if the EPEL repository is enabled 

	```
	yum repolist enabled
	```

4. Check if ocserv is available for install:  

	``` 
	yum info ocserv
	```

5. Install ocserv. This will install ocserv and its dependencies, including radcli radius libraries.

	```
	yum install ocserv  
	```

### Installation - Fedora


1. Check if ocserv is available for install:  

	``` 
	dnf info ocserv
	```

1. Install ocserv. This will install ocserv and its dependencies, including radcli radius libraries.

	```
	dnf install ocserv  
	```


### Conclusion and final notes
This concludes **Ocserv Installation - CentOS, RHEL, Fedora** recipe. At this point Openconnect server should be ready to be configured.  
If you want to learn more, you can find Ocserv recipes here: http://www.infradead.org/ocserv/recipes.html

## Ocserv Installation - Debian, Ubuntu
Author: Mauro Gaspari  


###Scope
This recipe provides step by step instructions on how to install ocserv repositories on Debian and Ubuntu operative systems.

### Platforms used for testing
This Recipe was tested on the following platforms:   

- Debian 8 (testing) on amd64 architecture
- Ubuntu 16.04 LTS on amd64 architecture


### Assumptions

- This recipe assumes the reader has a basic understanding of a GNU/linux system and all commands are run from a privileged user. It is recommended to login the system using root.If not possible, execute "su root" or "sudo su" to get highest privileges.


###Requirements
- During this recipe, Ubuntu users will be asked to check and enable "universe" repository. Please note that Ubuntu 16.04 LTS is the first ubuntu version that includes ocserv and radcli in its repositories. 
- At the time of writing the recipe, Debian 8 stable does not include ocserv and radcli packages. Jessie-Backports includes radcli but not ocserv. In order to install ocserv and radcli from Debian 8 repositories, it is necessary to be on testing or unstable. This recipe assumes user has testing repositories installed and enabled.
- Debian/Ubuntu package currently available on repositories does not support radius authentication. If radius authentication is required, it is recommended to skip this recipe and install from sources. Refer to http://www.infradead.org/ocserv/recipes-ocserv-installation-generic.html  and http://www.infradead.org/ocserv/recipes-ocserv-configuration-basic.html .  


### Details on lab used on this recipe
- network 192.169.5.0/24 (netmask 255.255.255.0)
- ocserv ip 192.168.5.254
- ocserv hostname fw01


### Installation - Debian (testing or unstable)

1. Check that Debian repositories are configured for either "testing" or "stretch". As previously mentioned, "unstable" or "sid" also works. 

	```
	cat /etc/apt/sources.list
	
	```

2. Update apt cache

	```
	apt-get update
	```

3. Check if ocserv is available for install:  

	``` 
	apt-cache show ocserv
	```

5. Install ocserv. This will install ocserv and its dependencies, however radcli will not be automatically installed. Currently Debian/Ubuntu ocserv package does not support radius authentication.

	```
	apt-get install ocserv  
	```

### Installation - Ubuntu (minimum version required 16.04 LTS)

1. Check that Ubuntu "universe" repository is enabled. Edit the file and uncomment repository if needed.

	```
	cat /etc/apt/sources.list
	
	```

2. Update apt cache

	```
	apt-get update
	```

3. Check if ocserv is available for install:  

	``` 
	apt-cache show ocserv
	```

5. Install ocserv. This will install ocserv and its dependencies, however radcli will not be automatically installed. Currently Debian/Ubuntu ocserv package does not support radius authentication.

	```
	apt-get install ocserv  
	```

### Conclusion and final notes
This concludes **Ocserv Installation - Debian, Ubuntu** recipe. At this point Openconnect server should be ready to be configured.  
If you want to learn more, you can find Ocserv recipes here: http://www.infradead.org/ocserv/recipes.html

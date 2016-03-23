## Ocserv Authentication - RADIUS (radcli)

Author: Mauro Gaspari  


###Scope
This Recipe provides step by step instructions on how to install, configure, and test RADIUS Authentication for Openconnect Server. This recipe focuses on generic installation instructions, from packages available on Openconnect server and radcli websites. No precompiled binary packages will be used, therefore this recipe applies to all linux distributions.  

### Platforms used for testing
This Recipe was tested on the following platforms: 
  
- Debian 8 (systemd) on armhf architecture.  
- Ubuntu Server 15.10 (systemd) on amd64 architecture.  
- Gentoo (openRC) on amd64 architecture.

### Assumptions

- This recipe assumes the reader has a basic understanding of a linux system and all commands are run from a privileged user. It is recommended to login the system using root.If not possible, execute "su root" or "sudo su" to get highest privileges.
- Software needed to compile from source is required, please refer to distribution specific guides to install needed packages. A few examples below:  
**Fedora:** yum install egcs cpp binutils glibc-devel egcs-c++ libstdc++  
**Debian:** apt-get install build-essential   
- Throughout this recipe, version number will be x.x.x . Replace x.x.x on all steps below with preferred version number.

###Requirements
Since version 0.10.6 ocserv RADIUS authentication depends on either libradius-client 1.1.7 or radcli libraries.
At the time of writing, many distributions offer libradius-client 1.1.6.  
In order to avoid unmet dependencies or radius libraries lagging behind ocserv in any way, Nikos Mavrogiannopulos decided to create radcli libraries.  
radcli libraries are updated to match ocserv requirements, very easy to install and use.
This recipe includes instructions to compile radcli libraries.  

**NOTE:** radcli (or libradius-client) need to be installed before ocserv. If ocserv is compiled before radius libraries, it will build without radius support. Even if radius libraries are compiled/installed afterwards, it will not work, and ocserv will return errors when you try to start the service with RADIUS authentication enabled.



### Details on lab used on this recipe
- network 192.169.5.0/24 (netmask 255.255.255.0)
- ocserv ip 192.168.5.254
- ocserv hostname fw01
- radius server ip 192.168.5.5  
- radius server password: radiustestsecretpassword


## Installation



### Radcli installation

1. Create a folder to contain downloaded software sources

	```
	mkdir /usr/local/src/radcli
	```

2. Move to radcli folder

	```
	cd /usr/local/src/radcli/
	```

3. Download radcli libraries

	```
	wget https://github.com/radcli/radcli/releases/download/x.x.x/radcli-x.x.x.tar.gz
	```

4. Decompress the archive

	```
	tar zxvf radcli-x.x.x.tar.gz
	```

5. Move to radcli-x.x.x folder

	```
	cd radcli-x.x.x
	```

6. Compile and install

	```
	./configure && make && make install
	```

That's it. Radcli is now installed. remember to keep a copy of source package in your /usr/local/src/radcli/ folder, so that you can easily remove the software if needed. If you need to uninstall, you can do it with the following commands:

	```
	cd /usr/local/src/radcli/radcli-x.x.x
	make uninstall
	```

### radcli configuration (No TLS)

1. Move to radcli folder

	```
	cd /usr/local/etc/radcli/
	```

2. Open radiusclient.conf with text editor

	```
	nano radiusclient.conf
	```

3. Configure radius settings according to your environment. Following lab details for this recipe, we will need to change the following paramters:

	```
	nas-identifier fw01  
	authserver 192.168.5.5  
	acctserver 192.168.5.5  
	servers /usr/local/etc/radcli/servers  
	dictionary /usr/local/etc/radcli/dictionary  
	default_realm  
	radius_timeout 10  
	radius_retries 3  
	bindaddr 192.168.5.254
	```

4. Save and exit (in nano, ctrl+o to save, then ctrl+x to exit)

5. Open servers file with text editor  

	```
	nano servers
	```

6. Configure servers settings according to your environment. Following lab details for this recipe we will need to add one line to servers file:

	```
	192.168.5.5 radiustestsecretpassword 
	```

7. Save and exit (in nano, ctrl+o to save, then ctrl+x to exit)

### Security considerations and radcli configuration with TLS. 

If you have security concerns, for example on a corporate network, there are several ways to secure radius communications. One very common way to achieve this, is by isolating vlans.  
If your RADIUS server requires TLS authentication of radius messages for security reasons, or if you are unable to implement separate management vlans, please use radiusclient-tls.conf instead of radiusclient.conf, and servers-tls instead of servers.
Radius configuration with TLS is out of the scope of this document. Having said that, comments on radiusclient-tls.conf and servers-tls are clear and a system admin should have no issue with configuration.


### Ocserv Installation
You can install ocserv using distribution packages. If ocserv is not available in your package manager, you can follow instructions on Ocserv installation recipe, available here: [Ocserv Installation - Generic](ocserv-installation-generic.md)

### Ocserv Configuration for Radius authentication
In order to enable radius authentication for ocserv, follow the steps below.  

1. move to ocserv folder. Note. if you installed from sources following    [Ocserv Installation - Generic](ocserv-installation-generic.md), ocserv folder is /usr/local/etc/ocserv/

	```
	cd /etc/ocserv
	```

2. open ocserv.conf with text editor  

	```  
	nano ocserv.conf
	```

3. comment all lines starting with "auth =", it should look like this:  

	```
	#auth = "pam"  
	#auth = "pam[gid-min=1000]"  
	#auth = "plain[passwd=./ocserv.passwd]"  
	#auth = "certificate"  
	#auth = "radius[config=/etc/radiusclient/radiusclient.conf,groupconfig=true]"  
	```

4. add a line as follows:  

	```
	auth = "radius [config=/usr/local/etc/radcli/radiusclient.conf,groupconfig=true]"
	```

5. If you need RADIUS accounting, add a line in the "Accounting methods available" Section:

	```  
	acct = "radius [config=/usr/local/etc/radcli/radiusclient.conf,groupconfig=true]"
	```

### Conclusion and final notes
This concludes **Ocserv Authentication - RADIUS (radcli)** recipe. At this point Openconnect server should be working with radcli RADIUS libraries.  
If you want to learn more, you can find Ocserv recipes here: http://www.infradead.org/ocserv/recipes.html
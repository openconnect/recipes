## Ocserv Configuration - Basic

Author: Mauro Gaspari  

### Scope
This recipe provides step by step instructions on how to configure ocserv for basic functionality.

### Platforms used for testing
This Recipe was tested on the following platforms: 
  
 - Debian 8 (systemd) on armhf architecture.  
 - Ubuntu Server 15.10 (systemd) on amd64 architecture.  
 - Gentoo (openRC) on amd64 architecture.
- Fedora 23

### Assumptions
 - This recipe assumes the reader has a basic understanding of a linux system and all commands
   are run from a privileged user. It is recommended to login the system using root. If not possible,
   execute "su root" or "sudo su" to get highest privileges.
 - The reader is applying ocserv to a linux server that is already configured as a router and has a
   firewall running (iptables, shorewall, or other).

### Requirements

### Network settings used on this recipe
 - network 192.169.5.0/24 (netmask 255.255.255.0)
 - ocserv ip 192.168.5.254
 - ocserv hostname fw01
 - authentication method used for testing: pam


### Certificate Management (Self Signed)

**Create CA template file and server template file:**

 1. Create a folder to store your certificates 
	```
	mkdir /root/certificates
	```

 2. Move to certificetes folder

	```  
	cd /root/certificates  
	```

 3. Create CA and server templates based on this example file, edit parameters according to your organization name and needs. Please note that anyconnect VPN clients connecting to your ocserv will complain if certificates do not match hostname, or if are self signed.  

	```
	nano ca.tmpl
	```

	```
	cn = "your organization’s certificate authority"  
	organization = "your organization"  
	serial = 1  
	expiration_days = 3650  
	ca  
	signing_key  
	cert_signing_key  
	crl_signing_key  
	```

 4. Create Server template (edit parameters according to your organization name and needs)  

	```
	nano server.tmpl
	```

	```
	cn = "a sever's name, usually matches hostname"  
	organization = "your organization"  
	serial = 2  
	expiration_days = 3650
	signing_key
	encryption_key
	tls_www_server
	dns_name = "your organization's host name"
	#ip_address = "if no hostname uncomment and set the IP address here"
	```

 5. Generate CA key, CA certificate:   

	```
	certtool --generate-privkey --outfile ca-key.pem
	certtool --generate-self-signed --load-privkey ca-key.pem --template ca.tmpl --outfile 	ca-cert.pem
	```

 6. Generate Server key and certificate  

	```
	certtool --generate-privkey --outfile server-key.pem
	certtool --generate-certificate --load-privkey server-key.pem --load-ca-certificate ca-cert.pem --load-ca-privkey ca-key.pem --template server.tmpl --outfile server-cert.pem
	```

 7. Copy certificates in ocserv directory

	```
	cp server-cert.pem server-key.pem /etc/ocserv/
	```

### Configure ocserv

1. Open /etc/ocserv/ocserv.conf file  

	```
	nano /etc/ocserv/ocserv.conf
	```

2. In the Authentication section, comment all lines and add the following line:  

	```
	auth = "pam"
	```

3. In the TCP and UDP port number, leave the default and make sure both lines are uncommented  

	```
	tcp-port = 443
	udp-port = 443  
	```

4. In the seccomp section, decide if you want to use seccomp or not. If you removed seccomp when compiling or did not install seccomp packages, disable seccomp or ocserv will fail to start.  

	```  
	isolate-workers = true  
	```

5. In the Network Settings section, change the following lines:  

	```
	ipv4-network = 192.168.5.254  
	ipv4-netmask = 255.255.255.0  
	dns = 8.8.8.8  
	```

6. In the "Routes to be forwarded to the client" section, commend all lines and add the following line:  

	```
	route = 192.168.5.0/255.255.255.0
	```

7. Save the file and exit (CTRL+o to save, CTRL+x to exit)  

### Start ocserv and test

To manually start ocserv:  

	```
	ocserv -c /etc/ocserv/ocserv.conf  
	```

Authentication was set to pam, so from your client you can use any linux users of your system

### Use ocserv as a service and enable service start on system boot


If you are using systemd, you can activate ocserv easily by doing the following: 

1. Copy systemd script

	```
	cp /usr/share/doc/ocserv/doc/systemd/standalone/ocserv.service /lib/systemd/system  
	```

2. Enable ocserv on system bootup  

	```
	systemctl enable ocserv.service  
	```

Note that scripts for other init systems are currently not included in ocserv package.

### Final notes

This concludes **Ocserv Configuration - Basic** recipe. At this point Openconnect server
should be ready to accept VPN connections. Remember to open ports on your firewall, and
test connection.   


## How to setup ocserv for PAM authentication

Author: Mauro Gaspari  


### Scope

This Recipe provides step by step instructions on how to configure and 
test PAM Authentication for Openconnect Server. No precompiled binary 
packages will be used, therefore this recipe applies to all linux 
distributions. As a secondary and optional scope, this recipe will show
how to integrate Webmin and Usermin with PAM authentication, to provide
end users a quick and easy way to reset their own passwords.

### Platforms used for testing

This Recipe was tested on the following platforms: 
  
 * Debian 8 (systemd) on armhf architecture.  
 * Ubuntu Server 16.04 (systemd) on amd64 architecture.  
 * Gentoo (openRC) on amd64 architecture.

### Assumptions

This recipe assumes the reader has a basic understanding of a linux
system and all commands are run from a privileged user. It is recommended
to login the system using root.If not possible, execute ```su root``` or
```sudo su``` to get highest privileges.

### Requirements

No special requirements to use PAM authentication on linux computers.


### Details on lab used on this recipe
 
 * network 192.169.5.0/24 (netmask 255.255.255.0)
 * ocserv ip 192.168.5.254
 * ocserv hostname fw01


### Ocserv Configuration for Radius authentication

In order to enable PAM authentication for ocserv, follow the steps below.  

1. Move to ocserv folder. Note. if you installed from sources following
	[Ocserv Installation - Generic](ocserv-installation-generic.md),
	the ocserv folder is at ```/usr/local/etc/ocserv/```.

	```
	cd /etc/ocserv
	```
2. Open ocserv.conf with text editor  
	```  
	nano ocserv.conf
	```
3. Comment all lines starting with "auth =", it should look like this:  
	```
	#auth = "pam"

	#auth = "pam[gid-min=1000]"  

	#auth = "plain[passwd=./ocserv.passwd]"  

	#auth = "certificate"  

	#auth = "radius[config=/etc/radiusclient/radiusclient.conf,groupconfig=true]"  
	```
4. Add a line as follows:  

	```
	auth = "pam"
	```
5. If you need to limit ocserv authentication to a specific set of users,
	you can change the non-commented line to the following:
	```  
	auth = "pam[gid-min=1000]"
	```
	This means that only account number 1000 and above can be authenticated
	via openconnect server. git-min number can be changed according to
	administrator needs.

## Optional - Install and use Webmin & Usermin to allow users to reset their own passwords

It is often required even in small to medium businesses, that users can
reset their password without any admin knowing them. In this scenario,
use of RADIUS is preferred as all passwords are in sync. However, for
many reasons it could be not possible or approved to use RADIUS. PAM
authentication advantage is that ocserv depends on no other systems to
provide authentication. It is also considered to be much easier to
configure and maintain compared to RADIUS.

Having said that, end users often do not know how to use ssh and linux
cli to reset their passwords. This is where Webmin and Usermin come to 
play. Users can  login from the LAN (or even the WAN if required) using
any web browser. Usermin will provide a very limited web gui where end
users can reset their passwords.

### Webmin Installation

Download and install webmin for your distribution. Webmin packages for
main distributions are [available on this page](http://www.webmin.com/download.html).

#### Debian

```
wget http://prdownloads.sourceforge.net/webadmin/webmin_x.xxx_all.deb
dpkg -i webmin_x.xxx_all.deb
```
In case dpkg returns dependency problem, run the following command:
```
apt-get -f install
```

#### Fedora

```
wget http://prdownloads.sourceforge.net/webadmin/webmin-x.xxx-x.noarch.rpm
rpm -I webmin-x.xxx-x.noarch.rpm
```

#### Gentoo

```
emerge webmin
```

At this point, webmin is installed. On its default settings, it is
reachable on its port 10000. In this case: https://192.168.5.254:10000  

A few of notes:  
- It is advisable not to use default ports.  
- If connection cannot be established, remember to open firewall ports.  
- It is advisable to open firewall ports only from the LAN.  

### Usermin Installation

Once Webmin is installed, Usermin installation is very easy:

1. Connect to your Webmin and login with your admin user.
2. On the left menu, click Un-used Modules.
3. Select "Usermin Configuration"
4. Click on "Install Usermin tar.gz package"

At this point, usermin is installed. On its default settings, it is
reachable on its port 20000. In this case: https://192.168.5.254:20000

A few of notes:  
- It is advisable not to use default ports.  
- If connection cannot be established, remember to open firewall ports.  
- It is advisable to open firewall ports only from the LAN.  

### Usermin Configuration

Once installed, usermin can be configured and upgraded by system
administrator via webmin - Usermin Configuration. 

### Usermin Usage

Users can connect to Usermin interface using any web browser, and perform
the functions allowed by system administrator. In this recipe we focused
on the most basic one, which is for users to login and reset their own
passwords.


### Conclusion and final notes

This concludes the ocserv **PAM** recipe. At this point
the Openconnect server should be working with PAM authentication.



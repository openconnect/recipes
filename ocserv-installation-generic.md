## Ocserv Installation - Generic

Author: Mauro Gaspari  


###Scope
This recipe provides step by step instructions on how to install ocserv from archives available on Openconnect server webpage. No precompiled binary packages will be used, therefore this recipe applies to all linux distributions. 

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
- The reader is applying ocserv to a linux server that is already configured as a router and has a firewall running (iptables, shorewall, or other).
- Throughout this recipe, version number will be x.xx.x . Replace x.xx.x on all steps below with preferred version number.

###Requirements
Openconnect VPN server (ocserv) depends on the following packages:

##### Debian/Ubuntu  
 
**Required Packages**  
GNUTLS: libgnutls28-dev  

**Recommended Packages**   

TCP Wrappers: libwrap0-dev  
PAM: libpam0g-dev  
LZ4: liblz4-dev  
seccomp: libseccomp-dev  
occtl: libreadline-dev, libnl-route-3-dev  
GSSAPI: libkrb5-dev  
OATH: liboath-dev    
libprotobuf-c0-dev  
libtalloc-dev  
libhttp-parser-dev  
libpcl1-dev  
libopts25-dev  
autogen  
protobuf-c-compiler  
gperf  
liblockfile-bin  
nuttcp  
libev4  
libev-dev  

**Installation**

```
apt-get install libgnutls28-dev libwrap0-dev libpam0g-dev liblz4-dev libseccomp-dev libreadline-dev libnl-route-3-dev libkrb5-dev liboath-dev libprotobuf-c0-dev libtalloc-dev libhttp-parser-dev libpcl1-dev libopts25-dev autogen protobuf-c-compiler gperf liblockfile-bin nuttcp libev4 libev-dev
```

##### Fedora/CentOS

**Required Packages**  
GNUTLS: gnutls-devel  

**Recommended Packages** 

TCP Wrappers: tcp_wrappers-devel  
PAM: pam-devel  
LZ4: lz4-devel  
seccomp: libseccomp-devel  
occtl: readline-devel, libnl3-devel  
GSSAPI: krb5-devel  
OATH: liboath-devel  
protobuf-c-devel  
libtalloc-devel  
http-parser-devel  
pcllib-devel  
autogen-libopts-devel  
autogen  
protobuf-c  
gperf  
lockfile-progs  
nuttcp  
uid_wrapper  
socket_wrapper  
gssntlmssp  

**Installation**  

```
yum install gnutls-devel tcp_wrappers-devel pam-devel lz4-devel libseccomp-devel readline-devel libnl3-devel krb5-devel liboath-devel protobuf-c-devel libtalloc-devel http-parser-devel pcllib-devel autogen-libopts-devel autogen protobuf-c gperf lockfile-progs nuttcp uid_wrapper socket_wrapper gssntlmssp
```

##### Gentoo

**Required Packages**  
GNUTLS: net-libs/gnutls  

**Recommended Packages**

TCP Wrappers: sys-apps/tcp-wrappers  
PAM: sys-libs/pam  
LZ4: app-arch/lz4 
seccomp: sys-libs/libseccomp  
occtl: sys-libs/readline, dev-libs/libnl  
GSSAPI: app-crypt/mit-krb5  
OATH: sys-auth/oath-toolkit
Nettle: dev-libs/nettle  
dev-libs/protobuf-c  
sys-devel/autogen  
net-libs/http-parser  
dev-util/gperf  
app-misc/lockfile-progs  
sys-libs/uid_wrapper  
net-libs/socket_wrapper  
dev-libs/libev  
dev-libs/libevdev  

**Installation**  

```
emerge --ask net-libs/gnutls sys-apps/tcp-wrappers sys-libs/pam app-arch/lz4 sys-libs/libseccomp sys-libs/readline dev-libs/libnl app-crypt/mit-krb5 sys-auth/oath-toolkit dev-libs/nettle dev-libs/protobuf-c sys-devel/autogen net-libs/http-parser dev-util/gperf app-misc/lockfile-progs sys-libs/uid_wrapper net-libs/socket_wrapper dev-libs/libev dev-libs/libevdev
```

### Details on lab used on this recipe
- network 192.169.5.0/24 (netmask 255.255.255.0)
- ocserv ip 192.168.5.254
- ocserv hostname fw01


### Installation
**NOTE for RADIUS users:**  In order to use RADIUS authentication, necessary RADIUS libraries must be installed before ocserv is compiled. Please refer to specific recipe: [Ocserv Authentication - RADIUS (radcli)](ocserv-authentication-radius-radcli.md) for detailed instructions on RADIUS libraries installation and configuration.  

**NOTE - SECCOMP:** If configuration fails, it could be an error with libseccomp version, or platform that does not support seccomp. In this case you might want to try to compile without seccomp support:
./configure --disable-seccomp && make && make install  

**NOTE - sources and installation directory** I recommend using the /usr/local/src/ocserv/ folder to store downloaded ocserv versions, and /usr/local/etc/ocserv/ for ocserv files and configurations. Keeping a copy here will use up some space, but will be useful to recompile or remove ocserv later on. Main reason to use /usr/local/ as base path, is to separate compiled software from other software installed via distribution package managers.

1. Create and move to /usr/local/src/ocserv folder 

	```
	mkdir /usr/local/src/ocserv
	cd /usr/local/src/ocserv
	```

2. Check available ocserv versions on: ftp://ftp.infradead.org/pub/ocserv/ 

3. Download ocserv from OpenConnect server:  

	```
	wget ftp://ftp.infradead.org/pub/ocserv/ocserv-x.xx.x.tar.xz
	```

4. Extract the archive:  

	``` 
	tar xvf ocserv-x.xx.x.tar.xz
	```

5. Move to ocserv-x.xx.x directory  

	```
	cd ocserv-x.xx.x  
	```

6. Compile ocserv  

	```
	./configure && make && make install
	```

7. Create /usr/local/etc/ocserv folder  

	```
	mkdir /usr/local/etc/ocserv
	```

8. copy configuration sample to /etc/ocserv folder  

	```
	cp /usr/share/doc/ocserv/doc/sample.config /usr/local/etc/ocserv/ocserv.config
	```

### Conclusion and final notes
This concludes **Ocserv Installation - Generic ** recipe. At this point Openconnect server should be ready to be configured.  
If you want to learn more, you can find Ocserv recipes here: http://www.infradead.org/ocserv/recipes.html

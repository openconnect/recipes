## Ocserv Certificates - letsencrypt

Author: Mauro Gaspari  

###Scope
This recipe provides a deployment example of letsencrypt to provide ssl certificates for ocserv.  
This recipe does not claim to be a step-by-step guide or a letsencrypt tutorial, as there are plenty of those available online. Also, this recipe does not claim to be the best or most secure letsencrypt setup, but barely a starting point example for a GNU/Linux based router/firewall with Ocserv.  

### Platforms used for testing  
This Recipe was tested on the following platforms: 
  
- Ubuntu Server 18.04LTS on amd64 architecture.  
- CentOS 7 on amd64 architecture.  
- openSUSE Tumbleweed on amd64 architecture.  


### Assumptions  
- This recipe assumes the reader has a basic understanding of a GNU/Linux system and all commands are run from a privileged user. It is recommended to login the system using root. If not possible, execute "su root" or "sudo su" to get highest privileges.  
- The reader is applying ocserv to a GNU/Linux server that is already configured as a router.  
### Requirements
- The GNU/Linux server in the recipe is configured to work as a router. sysctl.conf is already configured with the line:  
```
net.ipv4.ip_forward = 1  
```  
- The GNU/Linux server in the recipe has ocserv installed and configured. Please refer to <http://www.infradead.org/ocserv/recipes.html> if needed.  


### Details on lab used on this recipe
- network 192.169.5.0/24 (netmask 255.255.255.0)
- ocserv ip 192.168.5.254
- ocserv hostname fw01
- ocserv WAN interface is eth0
- ocserv LAN interface is eth1
- ocserv ports for openconnect vpn are default TCP 443 and UDP 443
- letsencrypt uses port TCP 80. This is done to avoid overlapping with TCP 443 used for ocserv




###Install let's encrypt to manage certificates  

**Debian/Ubuntu**  

```
apt-get install certbot
```  

**CentOS/RHEL/Fedora**  
On older versions  
```
yum install certbot  
```  
Or, on latest versions  
```
dnf install certbot  
```  

**openSUSE**  
```
zypper in shorewall certbot  
```  


###Create certificates for ocserv  
```
certbot certonly --standalone --preferred-challenges http --agree-tos --email your-email-address -d vpn.yourdomain.domain
```  
**NOTE1:** closely monitor the output to find any issue with certificate creation.  
**NOTE2:** make sure you have a valid dns A record to point "vpn.yourdomain.domain" to the public IP address configured of your wan.  

###Setup auto renewal of certificates  
Edit: /etc/crontab  
Add this line to your crontab:  
```
15 00 * * * root certbot renew --quiet && systemctl restart ocserv
```  

###Configure openconnect server  
Configure ocserv according to instructions on official site: <https://ocserv.gitlab.io/www/recipes-ocserv-configuration-basic.html>
- Skip the certificate creation step, we are getting certificates from letsencrypt.  
- Instead of generating certificates, link the location where your certificates are stored by letsencrypt: /etc/letsencrypt/live/vpn.yourdomain.domain  

###restart openconnect server  
```
service ocserv restart
```  
**Check status**  
```
service ocserv status
```  

###Firewall  
**Shorewall**  
Appendix A. of shorewall recipe has configuration examples for letsencrypt. Refer to the recipe here: <http://www.infradead.org/ocserv/recipes.html> .  


### Conclusion and final notes  
This concludes **Ocserv Certificates - letsencrypt** recipe. At this point Openconnect server should be configured with ssl certificates released by letsencrypt. Also, certificates will be automatically renewed with certbot.  
If you want to learn more, you can find Ocserv recipes here: <http://www.infradead.org/ocserv/recipes.html>

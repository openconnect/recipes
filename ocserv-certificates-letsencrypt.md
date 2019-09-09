## Ocserv Certificates - letsencrypt

Author: Mauro Gaspari  

###Scope
This recipe provides a deployment example of letsencrypt to provide ssl certificates for ocserv.   shorewall (ipv4) is used as firewall.  
This recipe does not claim to be a step-by-step guide or a letsencrypt tutorial, as there are plenty of those available online. Also, this recipe does not claim to be the best or most secure letsencrypt or shorewall setup, but barely a starting point example for a GNU/Linux based router/firewall with Ocserv.  

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
- The GNU/Linux server in the recipe has ocserv installed and configured with shorewall as firewall. Please refer to <http://www.infradead.org/ocserv/recipes.html> if needed.  


### Details on lab used on this recipe
- network 192.169.5.0/24 (netmask 255.255.255.0)
- ocserv ip 192.168.5.254
- ocserv hostname fw01
- ocserv WAN interface is eth0
- ocserv LAN interface is eth1
- ocserv ports for openconnect vpn are default TCP 443 and UDP 443
- letsencrypt uses port TCP 80. This is done to avoid overlapping with TCP 443 used for ocserv


### Details on Firewall configuration
- Basic shorewall rules should be configured prior to checking this recipe. If needed, please refer to <http://www.infradead.org/ocserv/recipes.html>  
- Traffic to firewall ports TCP 443 and UDP 443 is allowed from wan  
- Traffic to firewall port TCP 80 is allowed from wan, to allow letsencrypt updates. Note that firewall opens the port but web server is closed most of the time. web server will be brougt up only during certificate renewal.  
- An example to open port TCP 80 is provided, so that port TCP 80 is in stealth mode most of the time, and traffic allowed only during certificate renewal.  


###Configure shorewall firewall for ocserv  
- Add ocs interface by adding the below line to /etc/shorewall/interfaces  
```
ocs     OCS_IF      physical=vpns+
```  
- Add ocs  zone  by adding the below line to /etc/shorewall/zones  
```
ocs     ipv4
```  
- open ports by adding the below lines in shorewall rules file: /etc/shorewall/rules .  
- **NOTE1** port 80 is needed for letsencrypt. A temporary website will be enabled by certbot only during certificate creation or renewal port 80 therefore shows as closed from external port scans if you want to keep port 80 as stealth, see NOTE2.  
```
ACCEPT          net     $FW     tcp     80
ACCEPT          net     $FW     tcp     443
ACCEPT          net     $FW     udp     443
```  

- **NOTE2** if you prefer port 80 to be in stealth mode instead of showing up as closed from external port scans, use the rules in the example below. Note that with the rule below, http port accepts traffic only between 00:10 and 00:20. Port 80 will be in stealth mode unless during this time. And during this time it will be closed, unless letsencrypt opens it for a few seconds to renew certificates. Further tweaking such as day of the week, month, and more can be done in shorewall. Refer to official documentation <http://www.shorewall.net/manpages/shorewall-rules.html>.  Important part is that accept time matches crontab time for letsencrypt to match certificates.  
```
ACCEPT          net     $FW     tcp     80       -   -   -   -   -   -   localtz&timestart=00:10&timestop=00:20
ACCEPT          net     $FW     tcp     443
ACCEPT          net     $FW     udp     443
```  

- configure policies to allow VPN traffic between openconnect clients and devices in your local network by adding the below lines in shorewall policy file: etc/shorewall/policy  
```
loc     ocs     ACCEPT
ocs     loc     ACCEPT
```  
**NOTE3** if you do not want all traffic to be allowed between ocserv and local network, create specific rules in the /etc/shorewall/rules file instead.  

- check, restart, and save shorewall rules:  
```
shorewall check
shorewall restart
shorewall save
```  



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
**NOTE** See that letsencrypt renew time in crontab must be within shorewall timestart and timestop schedules if you are stealthing the port.



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

### Conclusion and final notes  
This concludes **Ocserv Certificates - letsencrypt** recipe. At this point Openconnect server should be configured with ssl certificates released by letsencrypt. Also, certificates will be automatically renewed with certbot.  
If you want to learn more, you can find Ocserv recipes here: <http://www.infradead.org/ocserv/recipes.html>


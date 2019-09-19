## Ocserv Firewall - shorewall IPv4

Author: Mauro Gaspari  

###Scope
This recipe provides a deployment example of shorewall (ipv4) for a GNU/Linux based router/firewall and ocserv as VPN server.  
This recipe does not claim to be a step-by-step guide or a shorewall tutorial, as there are plenty of those available online. Also, this recipe does not claim to be the best or most secure shorewall setup, but barely a starting point example for a GNU/Linux based router/firewall with Ocserv.  

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


### Details on Firewall configuration
- Traffic from LAN to WAN is allowed
- Traffic from LAN to Firewall is allowed
- Traffic from WAN to LAN is denied
- Traffic from WAN to Firewall is denied (exception of ocserv ports TCP/UDP 443
- Traffic from VPN to LAN is allowed
- Traffic from LAN to VPN is allowed
- Traffic from VPN to Firewall is allowed



### Disable distribution specific firewalls

**Ubuntu ufw**
```  
ufw disable 
```

**CentOS/RHEL/Fedora/openSUSE**  
  
```
systemctl stop firewalld
systemctl mask firewalld
```

### Install shorewall services  

**Debian/Ubuntu**  

```
apt-get install shorewall
```  

**CentOS/RHEL/Fedora**  
```
yum install iptables-services  
```  

**openSUSE**  

```
zypper in shorewall shorewall-docs
```  

### Shorewall configuration example  
Refer to the below examples for a functioning shorewall with ocserv.  
**Note for shorewall configuration:** Shorewall configurations are stored in a few files, usually in the folder /etc/shorewall . If the folder is empty, refer to your distribution shorewall man page, copy sample files from documentation folder to /etc/shorewall.  
You can also refer to official shorewall documentation: <http://www.shorewall.net/>  

**Shorewall master configuration file**  
Edit master configuration file and enable startup option: /etc/shorewall/shorewall.conf  
```
###############################################################################
#                      S T A R T U P   E N A B L E D
###############################################################################

STARTUP_ENABLED=Yes
```  



**Shorewall zones**  
Sample for /etc/shorewall/zones  
```
#------------------------------------------------------------------------------
# For information about entries in this file, type "man shorewall-zones"
###############################################################################
#ZONE   TYPE    OPTIONS                 IN                      OUT
#                                       OPTIONS                 OPTIONS
fw      firewall
net     ipv4
loc     ipv4
ocs     ipv4
```  

**Shorewall interfaces**  
Sample for /etc/shorewall/interfaces  
```
# For information about entries in this file, type "man shorewall-interfaces"
###############################################################################
?FORMAT 2
###############################################################################
#ZONE   INTERFACE       OPTIONS
net     NET_IF          dhcp,tcpflags,nosmurfs,routefilter,logmartians,sourceroute=0,physical=eth0
loc     LOC_IF          tcpflags,nosmurfs,routefilter,logmartians,physical=eth1
ocs     OCS_IF          physical=vpns+
```  

**Shorewall policy**  
Sample for /etc/shorewall/policy  
```
#------------------------------------------------------------------------------
# For information about entries in this file, type "man shorewall-policy"
###############################################################################
#SOURCE DEST            POLICY          LOGLEVEL        RATE    CONNLIMIT

loc     net             ACCEPT
loc     fw              ACCEPT
net     all             DROP            $LOG_LEVEL
fw      all             ACCEPT
loc     ocs             ACCEPT
ocs     loc             ACCEPT
ocs     fw              ACCEPT 
# THE FOLOWING POLICY MUST BE LAST
all     all             REJECT          $LOG_LEVEL
```  


**Shorewall rules**  
Sample for /etc/shorewall/rules  
```
#------------------------------------------------------------------------------
# For information about entries in this file, type "man shorewall-rules"
###############################################################################################################################################################>
#ACTION         SOURCE          DEST            PROTO   DEST    SOURCE          ORIGINAL        RATE            USER/   MARK    CONNLIMIT       TIME           >
#                                                       PORT    PORT(S)         DEST            LIMIT           GROUP
?SECTION ALL
?SECTION ESTABLISHED
?SECTION RELATED
?SECTION INVALID
?SECTION UNTRACKED
?SECTION NEW

#       Don't allow connection pickup from the net
#
Invalid(DROP)   net             all             tcp
#
#       Accept DNS connections from the firewall to the network
#
DNS(ACCEPT)     $FW             net
#
#       Accept SSH connections from the local network for administration
#
SSH(ACCEPT)     loc             $FW
#
#       Accept connections on port 443 (TCP and UDP) for OpenConnect Server
#
ACCEPT          all             $FW             tcp     443
ACCEPT          all             $FW             udp     443
#
#
#       PING SECTION
#
#       Allow Ping from the local network
#
Ping(ACCEPT)    loc             $FW
#
# Drop Ping from the "bad" net zone.. and prevent your log from being flooded..
#
Ping(DROP)      net             $FW
ACCEPT          $FW             loc             icmp
ACCEPT          $FW             net             icmp
```  

**Shorewall snat**  
Sample for /etc/shorewall/snat  
```
#------------------------------------------------------------------------------
# For information about entries in this file, type "man shorewall-snat"
#
# See http://shorewall.net/manpages/shorewall-snat.html for more information
###########################################################################################################################################
#ACTION                 SOURCE                  DEST            PROTO   PORT    IPSEC   MARK    USER    SWITCH  ORIGDEST        PROBABILITY
#
# Rules generated from masq file /home/teastep/shorewall/trunk/Shorewall/Samples/two-interfaces/masq by Shorewall 5.0.13-RC1 - Sat Oct 15 11:41:40 PDT 2016
#
MASQUERADE              10.0.0.0/8,\
                        169.254.0.0/16,\
                        172.16.0.0/12,\
                        192.168.0.0/16          NET_IF
```  

**Shorewall stoppedrules**  
Sample for /etc/shorewall/stoppedrules  
```
#------------------------------------------------------------------------------
# For information about entries in this file, type "man shorewall-stoppedrules"
###############################################################################
#ACTION         SOURCE          DEST            PROTO   DEST            SOURCE
#                                                       PORT(S)         PORT(S)
ACCEPT          LOC_IF          -
ACCEPT          -               LOC_IF
```  

###Test configuration, save and apply  
** Test configuration**  
Once configuration is completed, proceed and test it:
```
shorewall check
```  

**Start shorewall**  
If there are no errors in shorewall check, proceed and start/restart shorewall:
```
shorewall restart
```  

**Check shorewall status**  
```
shorewall status
```  

**Backup shorewall configuration**  
```
shorewall save
```  

**Enable shorewall to start on system boot**  
```
systemctl enable shorewall.service
```  

### Final Test  
In order to make sure everything is properly configured, a system reboot is recommended. Check that all services are started after boot, and that shorewall and ocserv are working as intended.

** Note for Webmin Users **  
Webmin users can enjoy web based shorewall management. 

### Security Note on IPS/IDS system
- Since Ocserv is the only exposed service on this server, a third party ISD/IPS system is not required. Ocserv includes client ban functionalities that can be easily customized. Please see ocserv.conf for more information, comments above each option are very clear.
- It is good practice, especially if admin wants to expose ssh, webmin, or other services to the WAN, to install and configure Fail2Ban package. Fail2Ban will automatically block source IPs when monitored services receive failed logins or suspicious actions.

### Conclusion and final notes
This concludes **Ocserv Firewall - shorewall IPv4** recipe. At this point shorewall will allow Openconnect server to receive VPN connections from the WAN interface.   
If you want to learn more, you can find Ocserv recipes here: <http://www.infradead.org/ocserv/recipes.html>

### Appendix A. Shorewall configuration for letsencrypt  

- Traffic to firewall port TCP 80 is allowed from wan, to allow letsencrypt updates. Note that firewall opens the port but web server is closed most of the time. web server will be brougt up only during certificate renewal.  
- An example to open port TCP 80 is provided, so that port TCP 80 is in stealth mode most of the time, and traffic allowed only during certificate renewal.  


#### Shorewall rules for letsencrypt - Standard rule  
- open ports by adding the below lines in shorewall rules file: /etc/shorewall/rules .  
- port 80 is needed for letsencrypt. A temporary website will be enabled by certbot only during certificate creation or renewal. Therefore port 80 shows as closed from external port scans. If you want to keep port 80 in stealth mode instead of closed, see paragraph below.  
```
ACCEPT          net     $FW     tcp     80
```  

#### Shorewall rules for letsencrypt - Stealth port rule  
If you prefer port 80 to be in stealth mode instead of showing up as closed from external port scans, use the rules in the example below. Note that with the rule below, http port accepts traffic only between 00:10 and 00:20. Port 80 will be in stealth mode outside of the 10 minutes specified in the rule. And even during the 10 minutes, the port will still be closed, unless letsencrypt opens it for a few seconds to renew certificates.  
Further tweaking such as day of the week, month, and more can be done in shorewall. Refer to official documentation <http://www.shorewall.net/manpages/shorewall-rules.html>.  **NOTE** Make sure that accept time matches crontab entry for letsencrypt certificates renewal.  
```
ACCEPT          net     $FW     tcp     80       -   -   -   -   -   -   localtz&timestart=00:10&timestop=00:20
```  

#### Restart shorewall to apply changes
- Once firewall rules are changed, do not forget to check, restart, and save shorewall rules:  
```
shorewall check
shorewall restart
shorewall save
```  

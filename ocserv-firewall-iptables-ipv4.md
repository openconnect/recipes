# Ocserv Firewall - iptables IPv4

Author: Mauro Gaspari  

## Scope

This recipe provides a deployment example of iptables (ipv4) for a GNU/Linux based router/firewall and ocserv as VPN server.  
This recipe does not claim to be a step-by-step guide or a iptables tutorial, as there are plenty of those available online. Also, this recipe does not claim to be the best or most secure iptables setup, but barely a starting point example for a GNU/Linux based router/firewall with Ocserv.

## Platforms used for testing

This Recipe was tested on the following platforms: 
  
- Debian 8 (systemd) on armhf architecture.  
- Ubuntu Server 16.04LTS (systemd) on amd64 architecture.  
- CentOS 7 (systemd) on amd64 architecture.
- Gentoo (openRC) on amd64 architecture.


## Assumptions

- This recipe assumes the reader has a basic understanding of a GNU/Linux system and all commands are run from a privileged user. It is recommended to login the system using root.If not possible, execute "su root" or "sudo su" to get highest privileges.
- The reader is applying ocserv to a GNU/Linux server that is already configured as a router.

## Requirements

- The GNU/Linux server in the recipe is configured to work as a router. sysctl.conf is already configured with the line:  
```net.ipv4.ip_forward = 1```  

- The GNU/Linux server in the recipe has ocserv installed and configured. Please refer to [installation recipes](README.md) if needed.


## Details on lab used on this recipe

- network 192.169.5.0/24 (netmask 255.255.255.0)
- ocserv ip 192.168.5.254
- ocserv hostname fw01
- ocserv WAN interface is eth0
- ocserv LAN interface is eth1
- ocserv ports for openconnect vpn are default TCP 443 and UDP 443
- Firewall is in learning mode on all 3 filtering chains. This means iptables is logging a lot of traffic. this could create problems with disk space or I/O on small devices, such as arm based mini-computers, or devices using flash memory for file system.  Learning mode is meant to be a temporary stage to learn and adjust the firewall. General practice is to create custom rules, place those before the LOG rules to reduce the logs. Once learning stage is completed, it is recommended to disable (comment) the LOG rules and change last rules from ACCEPT to DROP.  


## Details on Firewall configuration

### Filtering - input chain

- Firewall only allows connections to ports for OCSERV from WAN interface "eth0".  
- Firewall stateful rule is enabled.  
- Firewall allows incoming traffic from LAN interface "eth1" for minimal services. This includes DNS, DHCP, SSH, Webmin.  
- Firewall is in learning mode for all remaining traffic from LAN interface "eth1". This means all traffic is logged and then accepted. It is recommended to look at the logs marked as "IPTABLES-LOG-INPUT-LAN:", allow what is needed with specific rules, then change the rule that allow traffic from LAN to drop packets. LOG rule can be left enabled for a while so admin can  monitor and adjust as needed. This LOG rule can be enabled and disabled as needed.  
- Keep the last rule as drop all to avoid unwanted traffic from reaching the firewall.  

### Filtering - forward chain

- All traffic from LAN to WAN is allowed. This can be changed according to admin requirements or preferences. It  is usually ok to leave this rule in a SOHO environment if there are no strict rules to block outgoing traffic.  
- If the admin needs to create port forwards (DNAT) to internal server, remember to add firewall forwarding rules to accept the traffic. from WAN to LAN on DNAT ip address.  
- Firewall stateful rule is enabled.  
- Firewall is in learning mode for all remaining forwarded traffic. This means all traffic is logged and then accepted. It is recommended to look at the logs marked as "IPTABLES-LOG-FORWARD:", allow what is needed with specific rules, then change the last rule to drop packets. LOG rule can be left enabled for a while so admin can  monitor and adjust as needed. This log rule can be enabled and disabled as needed.    
- Change the last rule from ACCEPT to DROP once learning phase is completed, in order to avoid unwanted traffic to be forwarded.  

### Filtering - output chain

- Firewall stateful rule is enabled.  
- Firewall is in learning mode for all outgoing traffic. This means all traffic is logged and then accepted. It is recommended to look at the logs marked as "IPTABLES-LOG-OUTPUT:", allow what is needed with specific rules, then change the last rule to drop packets. LOG rule can be left enabled for a while so admin can  monitor and adjust as needed. This LOG rule can be enabled and disabled as needed.   
- Change the last rule from ACCEPT to DROP once learning phase is completed, in order to avoid unwanted outgoing traffic from firewall.  

### NAT - prerouting chain

- No rules in this chain. This is where admin needs to create port forwards (DNAT) if needed.  

### NAT - postrouting chain

- A single rule in this chain, is the generic outgoing NAT rule (Masquerade). 
- If the admin needs site to site VPNs, it is important to add rules to allow packets from local LAN to remote LAN. These rules must be before the generic outgoing NAT rule. Those are usually referred as "nat exemption" rules. 

## Enable Kernel Network Security options

1. Edit sysctl.conf  

	```nano /etc/sysctl.conf```

2. Add the following lines to sysctl.conf

	```
	# Protect from IP Spoofing  
	net.ipv4.conf.all.rp_filter = 1
	net.ipv4.conf.default.rp_filter = 1
	
	# Ignore ICMP broadcast requests
	net.ipv4.icmp_echo_ignore_broadcasts = 1
	
	# Protect from bad icmp error messages
	net.ipv4.icmp_ignore_bogus_error_responses = 1
	
	# Disable source packet routing
	net.ipv4.conf.all.accept_source_route = 0
	net.ipv6.conf.all.accept_source_route = 0
	net.ipv4.conf.default.accept_source_route = 0
	net.ipv6.conf.default.accept_source_route = 0
	
	# Turn on exec shield
	kernel.exec-shield = 1
	kernel.randomize_va_space = 1

	# Block SYN attacks
	net.ipv4.tcp_syncookies = 1
	net.ipv4.tcp_max_syn_backlog = 2048
	net.ipv4.tcp_synack_retries = 2
	net.ipv4.tcp_syn_retries = 5
	
	# Log Martians  
	net.ipv4.conf.all.log_martians = 1
	net.ipv4.icmp_ignore_bogus_error_responses = 1
	
	# Ignore send redirects
	net.ipv4.conf.all.send_redirects = 0
	net.ipv4.conf.default.send_redirects = 0
	
	# Ignore ICMP redirects
	net.ipv4.conf.all.accept_redirects = 0
	net.ipv6.conf.all.accept_redirects = 0
	net.ipv4.conf.default.accept_redirects = 0
	net.ipv6.conf.default.accept_redirects = 0
	net.ipv4.conf.all.secure_redirects = 0
	net.ipv4.conf.default.secure_redirects = 0
	```
 

3. Apply Changes without rebooting:  

	```sysctl -p```


## Disable distribution specific firewalls

 * Ubuntu UFW
```ufw disable```

 * CentOS/RHEL/Fedora
```
systemctl stop firewalld
systemctl mask firewalld
```

## Install and enable iptables services

 * Debian/Ubuntu
```
apt-get install iptables-persistent
service iptables start
```

 * CentOS/RHEL/Fedora
```
yum install iptables-services  
service iptables start
```

 * Gentoo
```
emerge --ask net-firewall/iptables 
rc-update add iptables default   
```

## Saved rules

For easy reference, this is where distribution specific iptables services save iptables rules.

 * Debian/Ubuntu
```
/etc/iptables/rules.v4
```

 * CentOS/RHEL/Fedora
```
/etc/sysconfig/iptables
```

 * Gentoo
```
/var/lib/iptables/rules-save  
```

**Note for Webmin Users **. Webmin users can enjoy web based iptables management. However, the basic configuration of webmin iptables module, called "Linux Firewall", under "Networking" has its own way of starting, saving, and restoring iptables rules.  
Default webmin ipv4 rules location is:  
```
/etc/iptables.up.rules
```

There are two easy ways to avoid conflicts and issues:  

 1. Use Webmin iptables management instead of installing distribution specific iptables services.  
 
 2. In order to coexist with distribution specific iptables services, it is recommended to change the webmin "Linux Firewall" "module config" options to match distribution specific iptables location. Please also note on main "Linux Firewall" page, keep the "Activate at boot" option to no, as distribution specific services will take care of this.


## iptables basic configuration

As already stated in the recipe's scope, this is not an ultimate firewall configuration, just a starting point to have a working firewall with common policies. There are no port forwards and the only traffic allowed from outside is to reach openconnect server, installed on same box.

 1. Copy the following in your firewall configuration file.

	```
	*nat
	:INPUT ACCEPT [0:0]
	:PREROUTING ACCEPT [0:0]
	:OUTPUT ACCEPT [0:0]
	:POSTROUTING ACCEPT [0:0]
	# Generic NAT for LAN Network 192.168.5.0/24  
	-A POSTROUTING -s 192.168.5.0/24 -o eth0 -j MASQUERADE
	COMMIT
	
	*mangle
	:PREROUTING ACCEPT [0:0]
	:INPUT ACCEPT [0:0]
	:FORWARD ACCEPT [0:0]
	:OUTPUT ACCEPT [0:0]
	:POSTROUTING ACCEPT [0:0]
	COMMIT
	
	*filter
	:INPUT ACCEPT [0:0]
	:FORWARD ACCEPT [0:0]
	:OUTPUT ACCEPT [0:0]
	# START INPUT RULES
	# Stateful Rule - INPUT
	-A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
	# ACCEPT traffic from Loopback interface
	-A INPUT -i lo -j ACCEPT
	# ACCEPT SSH from LAN
	-A INPUT -p tcp -m tcp -i eth1 --dport 22 -j ACCEPT
	# ACCEPT DHCP from LAN
	-A INPUT -p udp -m udp -i eth1 --dport 67:68 -j ACCEPT
	# ACCEPT Webmin from LAN (Optional, only for Webmin users)
	-A INPUT -p tcp -m tcp -i eth1 --dport 10000 -j ACCEPT
	# ACCEPT DNS UDP From LAN
	-A INPUT -p udp -m udp -i eth1 --dport 53 -j ACCEPT
	# ACCEPT DNS TCP From LAN
	-A INPUT -p tcp -m tcp -i eth1 --dport 53 -j ACCEPT
	# ACCEPT ping from LAN
	-A INPUT -p icmp --icmp-type echo-request -i eth1 -j ACCEPT
	# ACCEPT OpenConnect TCP From WAN
	-A INPUT -p tcp -m tcp -i eth0 --dport 443 -j ACCEPT
	# ACCEPT OpenConnect UPD From WAN
	-A INPUT -p udp -m udp -i eth0 --dport 443 -j ACCEPT
	# DROP wan traffic
	-A INPUT -i eth0 -j DROP
	# LOG LAN
	-A INPUT -i eth1 -j LOG --log-prefix "IPTABLES-LOG-INPUT-LAN:" --log-level 4
	# ACCEPT LAN traffic - Learning rule - Should be changed to DROP once custom rules are created.
	-A INPUT -i eth1 -j ACCEPT
	# LAST RULE - DROP all traffic
	-A INPUT -j DROP
	# END INPUT RULES
		
	# START FORWARD RULES
	# Stateful Rule - FORWARD
	-A FORWARD -m state --state ESTABLISHED,RELATED -j ACCEPT
	# ACCEPT LAN to WAN
	-A FORWARD -s 192.168.5.0/24 -j ACCEPT
	# LOG Forwarded traffic
	-A FORWARD -j LOG --log-prefix "IPTABLES-LOG-FORWARD:" --log-level 4
	# LAST RULE - ACCEPT all traffic - Should be changed to DROP once custom rules are created.
	-A FORWARD -j ACCEPT
	# END FORWARD RULES
	
	# START OUTPUT RULES
	# Stateful Rule - OUTPUT 
	-A OUTPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
	# LOG Outgoing traffic
	-A OUTPUT -j LOG --log-prefix "IPTABLES-LOG-OUTPUT:" --log-level 4
	# LAST RULE - ACCEPT all traffic - Should be 	changed to DROP once custom rules are created.
	-A OUTPUT -j ACCEPT
	# END OUTPUT RULES
	COMMIT
	```

 2. Check and apply rules
It is recommended to have a look at the rules, and tweak them for specific needs before applying. Worth of notice, if ssh and webmin ports are not standard, firewall input rules should be changed to match. Failing to do so will result in admin being locked out.  
Once all rules are reviewed and changed as needed, admin can proceed and apply. the general command is:  

	```iptables-restore < /path/to/your/iptables/rules/file```

### Examples

A few examples are given below.

 * Debian/Ubuntu
```
iptables-restore < /etc/iptables/rules.v4
```

 * CentOS/RHEL/Fedora
```
iptables-restore < /etc/sysconfig/iptables
```

 * Gentoo
```
iptables-restore < /var/lib/iptables/rules-save  
```

**Note for webmin users**  
Webmin users can apply rules from web interface.


## Security Note on IPS/IDS system

- Since Ocserv is the only exposed service on this server, a third party ISD/IPS system is not required. Ocserv includes client ban functionalities that can be easily customized. Please see ocserv.conf for more information, comments above each option are very clear.
- It is good practice, especially if admin wants to expose ssh, webmin, or other services to the WAN, to install and configure Fail2Ban package. Fail2Ban will automatically block source IPs when monitored services receive failed logins or suspicious actions.


## Final notes

This concludes **Ocserv Firewall - iptables IPv4** recipe. At this point iptables will allow Openconnect server to receive VPN connections from the WAN interface.

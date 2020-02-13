# Recipes for Openconnect VPN

This document contains recipes for various advanced configuration
settings in OpenConnect VPN server.

It is open for contribution; if you think you have a good overview
of a common (or not so-common) scenario, open a pull request
and submit it [at github](https://github.com/openconnect/recipes).

0. Installation
  * [Generic](ocserv-installation-generic.md)
  * [Centos/RHEL/Fedora](ocserv-installation-CentOS-RHEL-Fedora.md)
  * [Debian/Ubuntu](ocserv-installation-Debian-Ubuntu.md)
1. Generic recipes
  * [Basic ocserv configuration](ocserv-configuration-basic.md)
  * [Certificates - Letsencrypt](ocserv-certificates-letsencrypt.md)
  * [Firewall setup](ocserv-firewall-iptables-ipv4.md)
  * [Firewall setup with shorewall](ocserv-firewall-shorewall-ipv4.md)
 2. Authentication
  * [Two factor authentication with ocserv](ocserv-2fa.md)
  * [How to setup ocserv for RADIUS authentication](ocserv-authentication-radius-radcli.md)
  * [How to setup ocserv for PAM authentication](ocserv-authentication-pam.md)
  * [Using Kerberos authentication with ocserv](ocserv-kerberos.md)
  * [Integrating ocserv with FreeIPA](ocserv-freeipa.md)
  * [Integrating ocserv with Microsoft AD](ocserv-ad-authentication.md)
  * [How to restrict authentication by country](ocserv-country-blocking.md)
3. Networking
  * [Pseudo-Bridge setup with Proxy ARP](ocserv-pseudo-bridge.md)
  * [How to share the same port for VPN and HTTP](ocserv-multihost.md)
  * [Site to site links with ocserv](ocserv-site-to-site.md)
  * [VoIP network](ocserv-ip-phone.md)
4. Configuration Management
  * [Link to Ansible role](https://github.com/aprt5pr/lansible-role-ocserv)
  * [Link to Chef cookbook](https://supermarket.chef.io/cookbooks/ocserv)

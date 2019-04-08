# Pseudo-Bridge setup with Proxy ARP

Author: Mauro Gaspari

Proxy ARP allows to merge the openconnect VPN client network with
an existing network on your firewall/router. This configuration has
several advantage for both SOHO and enterprise environments.

## Advantages
Simpler network configuration, less routing, firewall rules to apply
and maintain. Easier to reach Multimedia Streaming software, such as Plex servers.
Easier networking for gamers.

In enterprise environments, there might be site-to-site IPSEC VPNs
connecting several offices, some of which could be out of our control
(another company handling far end of the tunnels). Since a remote VPN
clients IP are in the same subnet as local LAN computers, there is no
need to create special IPSEC Phase2s, Routing policies, Firewall
rules.

## Prerequisites
In order to take advantage of this setup, Proxy Arp must be enabled.
This can be done either as a global setting for all interfaces, or on
specific interfaces. The bare minimum is to enable it on the interface
that has the local IP addresses to be shared with OpenConnect clients.

To enable Proxy ARP on all interfaces, edit /etc/sysctl.conf with your
preferred editor and add the following lines:

```
# Enables Proxy ARP on all interfaces
net.ipv4.conf.all.proxy_arp   = 1
```

Save the file and exit. items in sysctl.conf are applied on system
boot. To apply this immediately, run the following command:

```
# sysctl -p
```

To enable Proxy ARP on a specific interface, edit /etc/sysctl.conf
with your preferred editor and add the following lines:

```
# Enables Proxy ARP on a specific interface (Replace "interfacename"
# with your interface, eg.eth0, eth0.100, wlan0, etc.)
net.ipv4.conf.interfacename.proxy_arp  = 1
```

Save the file and exit. items in sysctl.conf are applied on system
boot. To apply this immediately, run the following command:

```
# sysctl -p
```

Once this is done, we can proceed with OpenConnect Server configuration.
To avoid conflicts with IP addresses of the local network, and have a
better control of the IP addresses OpenConnect server uses, it is
advisable to use a subnet that is smaller than the main local subnet.
On this matter, it is also worth of note to mention that ocserv
virtual interface uses the first available address in assigned subnet.
so if local lan has 192.168.0.0/24 and ocserv ips are configured with
the same, 192.168.0.0/24, IP 192.168.0.1 will be used by ocserv
virtual interface. As this IP is often used by routers or servers, it
is likely to cause some problem of IP clashes.

## Example
- Local LAN subnet 192.168.0.0/24  (256 hosts)
- ocserv assigned subnet: 192.168.0.32/27 (32 hosts)

With this setup, OpenConnect network is considered a segment of the
Local LAN subnet. All routing, policies, nat, etc that apply to
192.168.0.0/24, also apply to ocserv assigned subnet 192.168.0.32/27.
OpenConnect can assign 30 addresses to server's virtual interface and clients.
OpenConnect server Virtual Interface IP will be 192.168.0.33

To avoid conflicts, make sure local LAN does not have any IP assigned
in this range: 192.168.0.32-63.

Also remember to check that DHCP is not going to assign leases in this
range: 192.168.0.32-63, or alternatively, that the ping-leases option to
ocserv's configuration is set to true.


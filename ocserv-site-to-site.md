# Site to site links

Author: Nikos Mavrogiannopoulos

In this scenario we describe a VPN server which provides multiple subnets
to connecting users, and some of these subnets are routed by some of
the users themselves. That is, a simple to setup site to site link.
For simplicity we examine an IPv4 setup like the following. Modification
for other setups should be trivial.

```

       10.10.1.0/24
       domain: .site1.com
       ------------
       |   Site1  |
       ------------
       /          \
      /            \
 ------------     ------------
 |   Site2  |     |  Client1 |
 ------------     ------------
 10.100.2.0/24
 domain: .site2.com

```

In that scenario Site1 advertises the 10.10.1.0/24, and we would like it
to route 10.10.2.0/24 through 'Site2' when they are connected.


## Prerequisites

The following instructions require ocserv 0.10.10.

## Configuration

### Site1

In order to enable this setup the server must have preconfigured the
routes that each client will serve. This can be done with the following
configuration settings in ocserv.conf of Site1.

```
split-dns = site1.com
split-dns = site2.com
ipv4-network = 192.168.1.0/24

route = 10.10.1.0/24

config-per-user = /etc/ocserv/config-per-user/

expose-iroutes = true
```

The 'ipv4-network' entry contains the addresses that the VPN clients will
be assigned to. The 'route' entry adds the routes that are directly routed by
the VPN server. The 'config-per-user' option instructs ocserv to read for
each user connecting the additional configuration file placed in 
```/etc/ocserv/config-per-user/``` which has the name of the user. If the
user is named 'Site2' ocserv will open ```/etc/ocserv/config-per-user/Site2```.

The 'expose-iroutes' option instructs ocserv to expose/advertise any 'iroute'
options found in the per-user configuration files to all connecting clients
(except the one serving it).

Hence, in our scenario the Site2 file will contain the following.
```
iroute = 10.100.2.0/24
```

This will instruct ocserv to setup route 10.100.2.0/24 via Site2 once it connects.

We skipped intentionally the 'split-dns' options in the config file above. For
the connecting VPN clients to be able to resolve both .site1.com and .site2.com sites 
the Site1 DNS server must be instructed to contact the Site2 DNS server for
the '.site2.com' addresses. That configuration is DNS-server specific. In
dnsmasq, for example, that can be achieved by adding a configuration line such as
"server=/site2.com/10.10.2.1" in its config file.

### Site2

Site2 will be a typical openconnect client. It will only need to allow
forwarding to and from the routes of Site1 (i.e., 10.10.1.0/24) and to
and from VPN client addresses (i.e., 192.168.1.0/24).

### Client1

No special configuration is needed for any of the openconnect clients.

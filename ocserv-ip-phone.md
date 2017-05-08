# VoIP network with ocserv

Author: Nikos Mavrogiannopoulos

In this scenario we describe a VPN server which is setup to provide
access to an existing VoIP network, i.e., to  a SIP server (e.g., asterisk or freeswitch).
We will not get into details of setting up the SIP server; we assume it is there
and it works. The VPN server will allow remote users to access the
SIP server in a secure and efficient way for VoIP telephony.
We assume the following setup.

```

       10.10.1.20
       server: sip.example.com
       ------------
       |SIP Server|
       ------------

       10.10.1.0/24
       domain: .example.com
       name: vpn.example.com
       ------------
       |VPN Server|
       ------------
       /            \
      /              \
 ----------------    ---------------
 |OpenWRT router|    |VPN/SIP Phone|
 ----------------    ---------------
      |
      |
 -----------
 |SIP Phone|
 -----------
```

## Prerequisites

The following instructions require at least ocserv 0.11.15. Two setups
will be presented. The first will use OpenWRT as an intermediate to access
the SIP server over VPN, while the second will use SIP phones which include
support for OpenConnect.

For the first setup, the hardware that is needed is:

 * [An OpenWRT router](https://wiki.openwrt.org/toh/start)
 * Any SIP phone

While the latter setup requires:

 * CISCO SPA525G or SPA525G2 (these models include an OpenConnect client)


## Configuration

### VPN server

Nothing particularly interesting is needed on the server setup. The
server only needs to provide access to the SIP server. For
authentication, it is recommended to allow password authentication
(either via radius, PAM or plain), since it will simplify the setup of the
clients described in the next sections.

```
split-dns = site1.com
ipv4-network = 172.17.163.0/24

route = 10.10.1.0/24
```

The 'ipv4-network' entry contains the addresses that the VPN clients will
be assigned to. The 'route' entry adds the routes that are directly routed by
the VPN server; in our case it must route to the SIP server.

Some hints for the rest of configuration options which relate to VoIP,
follow. It is recommended to set ```compression = false``` since attempting
to compress will increase latency which is undesirable for VoIP. If
compression is required, utilize the ```no-compress-limit``` option and
adjust it to the average size of RTP packets across the network. The default
value of 256-bytes should be sufficient for common codecs.

The default value of ```output-buffer``` in ocserv is adjusted for an average
performance in VoIP and throughput (downloading). If you experience latency
issues, you may want to experiment with different values. The lower the
value of ```output-buffer```, the lower the latency, as well as throughput.

The hardware of the server is also important for latency. If the server
provides hardware accelerated AES (e.g., x86-64 with AES-NI). It is best
to ensure that the version of gnutls used in the server includes
acceleration for the particular hardware. A way to check is the following.
```
$ GNUTLS_DEBUG_LEVEL=4 gnutls-cli -v
```

The output will be something like:
```
gnutls[2]: Enabled GnuTLS 3.5.5 logging...
gnutls[2]: getrandom random generator was detected
gnutls[2]: Intel SSSE3 was detected
gnutls[2]: Intel AES accelerator was detected
gnutls[2]: Intel GCM accelerator (AVX) was detected
```

### VPN/SIP Phone setup

#### Setting up VPN on the phone

The CISCO SPA525G series of phones can only be configured with a fixed
username-password pair, and as such the server must be configured to allow
plain username and password combinations.

To set it up, on the phone menu (after clicking the button that looks like a 'file') select:
Network configuration -> VPN

Then on the ```VPN Server``` field set the VPN server's external name
(e.g., vpn.example.com), and then set the ```User name``` and ```Password
fields```. For servers that are available in alternative to 443 ports, it
may be possible to set the server name as vpn.example.com:PORT, though I
have not test this setup.

Then select ```Enable Connection``` and if that works check ```Connect on bootup```.

#### Setting up SIP on rhw phone

Once the VPN connection is established on the phone you should setup the SIP
accounts. These are available through the web (administrator) interface. As
this information depends on the SIP server we skip that step.



### OpenWRT Router setup

#### Setting up OpenWRT

There are two ways to setup OpenWRT as an openconnect VPN client. Via the
```luci-proto-openconnect``` package or manually via the the
```openconnect``` package. In this section we describe the former method via
the web interface. For the manual configuration see
[the openconnect package documentation](https://github.com/openwrt/packages/tree/master/net/openconnect).

We assume basic knowledge of OpenWRT and accessing its web interface.

After installing the ```luci-proto-openconnect``` package, go to the web
interface at the Network -> Firewall menu. Click on the 'Add new zone'
option, name it something related to VPN and enable forwarding to and from
the LAN zone.

Then save and switch to the Network -> Interfaces menu. At this page
click the 'Add new interface' option and you will be redirected to a new
page. There you must name the interface (use something related to VPN), and
on the Protocol field select the 'OpenConnect (CISCO Anyconnect
compatible)' option.

After submitting you will be asked to set the VPN server name
(vpn.example.com), port, which typically is 443, and username and
password of the VPN user. You must also fill the 'VPN Server's certificate SHA1
hash' option. This can be done the following way.
One your Linux PC type the following commands.
```
$ /usr/sbin/openconnect vpn.example.com
.
.
.
Certificate from VPN server "vpn.example.com" failed verification.
Reason: signer not found
Enter 'yes' to accept, 'no' to abort; anything else to view: 
```

Type enter and you will see more information about the server's certificate.
What you need is the server's key hash which is displayed on the last lines.
```
.
.
.
Server key hash: sha1:6c4a751c55b59756fbcdb0de0a27598b0b34be48
```

Then copy paste the ```sha1:6c4a751c55b59756fbcdb0de0a27598b0b34be48``` to
the 'VPN Server's certificate SHA1 hash' field. If you are an administrator
making instructions for users to setup their routers, a more secure alternative
is to use the command ```certtool --key-id --infile server-cert.pem``` on
the server certificate, and provide the output value (with 'sha1:' prepended
to it), to the users setting OpenWRT up.

After entering all the information above on the 'General Setup' tab click
on the firewall settings, select the zone previously created, and click
'Save and Apply'.

If everything was entered correctly, you should be able to ping the SIP server's
internal IP, i.e., ```ping sip.example.com``` or ```ping 10.100.1.20```
should work.

#### Setting up the SIP phone

Once the VPN connection is established on the OpenWRT router any SIP phone
connected behind the OpenWRT router should be able to connect to the SIP
server on ```sip.example.com``` as if it was on the same LAN.


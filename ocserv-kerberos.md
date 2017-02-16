# Using Kerberos authentication with ocserv

Author: Nikos Mavrogiannopoulos	


## Introduction
One of the main features of the 0.10.x branch of OpenConnect VPN is the addition of
MS-KKDCP support and GSSAPI authentication. Putting the acronyms aside that means
that authentication with Kerberos, is greatly simplified for VPN users. A more
elaborate introduction is available at [this blog post](https://securityblog.redhat.com/2015/06/17/single-sign-on-with-openconnect-vpn-server-over-freeipa/).

## Setting the server up

It is required to instruct ocserv to allow GSSAPI for authentication. That can
be done with the following two lines in ocserv.conf.

```
auth = pam
enable-auth = gssapi[keytab=/etc/ocserv/key.tab,tgt-freshness-time=360]
```

That will allow authentication with Kerberos tickets, as well as with their password
(e.g., for clients that cannot obtain a ticket – like clients in mobile phones). 
In addition the 'keytab' option sets the ocserv's shared key with KDC
explicitly. The option ‘tgt-freshness-time’, when used, specifies the valid for VPN authentication 
lifetime, in seconds, of a Kerberos (TGT) ticket. A user will have to reauthenticate if this
time is exceeded. In effect that prevents the usage of the VPN for the whole lifetime of
a Kerberos ticket.

The following line will enable the MS-KKDCP proxy on ocserv. You’ll need to replace
the KERBEROS.REALM with your realm and the KDC IP address. That proxy will allow
the client to obtain Kerberos tickets through ocserv. The latter is an optional step if
your clients can obtain the tickets with other means.

```
kkdcp = /KdcProxy KERBEROS.REALM tcp@KDC-IP-ADDRESS:88
```

Note, that for PAM authentication to operate you will also need to set up a
/etc/pam.d/ocserv. We recommend to use pam_krb5 or pam_sssd for that, although
it can contain anything that best suits the local policy. An example for an SSSD
PAM configuration is shown in the [Fedora Deployment guide](https://docs.fedoraproject.org/en-US/Fedora/15/html/Deployment_Guide/chap-SSSD_User_Guide-Setting_Up_SSSD.html).


## Setting the client up

At the client side you must make sure you use openconnect 7.05 or later.
In order to use the KKDCP proxy as setup above, you need to setup Kerberos to use
ocserv as KDC. For that you’ll need to modify /etc/krb5.conf to contain the following:

```
[realms]
KERBEROS.REALM = {
    kdc = https://ocserv.example.com/KdcProxy
    http_anchors = FILE:/path-to-your/ca.pem
    admin_server = ocserv.example.com
    auto_to_local = DEFAULT
}

[domain_realm]
.kerberos.test = KERBEROS.REALM
kerberos.test = KERBEROS.REALM
```

Note that, ocserv.example.com should be replaced with the DNS name of your server,
and the /path-to-your/ca.pem should be replaced by the a PEM-formatted file which
holds the server’s Certificate Authority. For the KDC option the server’s DNS name
is preferred to an IP address to simplify server name verification for the Kerberos
libraries. At this point you should be able to use kinit to authenticate and obtain
a ticket from the Kerberos Authentication Server.

Note however, that kinit is very brief on the printed errors and a server certificate
verification error will not be easy to debug. Ensure that the http_anchors file is in
PEM format, it contains the Certificate Authority that signed the server’s certificate,
and that the server’s certificate DNS name matches the DNS name setup in the file.

Then, at a terminal run:

```
$ kinit
```

If the command succeeds, the ticket is obtained, and at this point you will be able to
setup openconnect from network manager GUI and connect to it using the Kerberos
credentials. To setup a VPN via NetworkManager on the system menu, select VPN, Network
Settings, and add a new Network of “CISCO AnyConnect Compatible VPN (openconnect)”. On
the Gateway field, fill in the server’s DNS name, add the server’s CA certificate, and
that’s all required.

To use the command line client with Kerberos the following trick is recommended. That
avoids using sudo with the client and runs the openconnect client as a normal user, after
having created a tun device. The reason it avoids using the openconnect client with sudo,
is that sudo will prevent access to the user’s Kerberos credentials.

```
# sudo ip tuntap add vpn0 mode tun user my-user-name
$ openconnect server.example.com -i vpn0
```


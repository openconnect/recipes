# Integrating ocserv with [FreeIPA](https://www.freeipa.org)

Author: Nikos Mavrogiannopoulos	


## Introduction
FreeIPA is an identity and policy management solution for POSIX based systems
based on Kerberos. This document provides instructions to take advantage of the
FreeIPA infrastructure when setting up ocserv. Such a combination simplifies
the management of properties such as second factor authentication, either
via passwords, or certificates. In addition it can be used to achieve
[single sign-on](https://securityblog.redhat.com/2015/06/17/single-sign-on-with-openconnect-vpn-server-over-freeipa/).

Note that, it is recommended to have the VPN server on a separate system than the
FreeIPA master server, and the following instructions make that assumption.

## Setting the server up

### Setting up the Kerberos principal

It is needed to add a new Principal into Kerberos to be used by the VPN server.
As OpenConnect VPN presents HTTPS end-point, a recommended Kerberos principal
should be HTTP/server. If for example, your server is 'vpn.example.com' you may
use the following instructions.

```
ipa service-add HTTP/vpn.example.com
ipa-getkeytab -s vpn.example.com -p HTTP/vpn.example.com -k /etc/ocserv/key.tab
```

These commands also extract the keytab to be used by ocserv and store it in
/etc/ocserv/key.tab. That contains the Kerberos keys relevant to that principal.


### Setting up the server certificate

FreeIPA acts a CA for its clients, so it is natural to sign the server's
certificate with it. The following commands generate a server certificate request
and pass it to FreeIPA for signing.

```
certtool --generate-privkey --outfile /etc/ocserv/server-key.pem
cat << _EOF_ >server.tmpl
  cn = "vpn.example.com"
  signing_key
  tls_www_server
  encryption_key
_EOF_
certtool --generate-request --load-privkey /etc/ocserv/server-key.pem \
	--outfile server.csr --template server.tmpl
ipa cert-request --principal HTTP/vpn.example.com server.csr
ipa service-show HTTP/vpn.example.com --out=/etc/ocserv/server-cert.pem
```

To enable the certificates generated you'll need to set the following
two lines in /etc/ocserv/ocserv.conf.
```
server-cert = /etc/ocserv/server-cert.pem
server-key = /etc/ocserv/server-key.pem
```

To automatically renew the server certificate when it expires the following
command is required.
```
ipa-getcert request -f /etc/ocserv/server-cert.pem -k /etc/ocserv/server-key.pem -r
```


### Setting up OpenConnect server

To enable logins with either Kerberos tickets or passwords checked via PAM with SSSD, it is required to
instruct ocserv to use PAM and  GSSAPI for authentication. That can be done with the following two lines in /etc/ocserv/ocserv.conf.

```
auth = pam
enable-auth = gssapi[keytab=/etc/ocserv/key.tab,require-local-user-map=true,tgt-freshness-time=360]
```

That will set the primary authentication method based on PAM, and make Kerberos (via GSSAPI)
a sufficient for login authentication method.
It is also important to modify /etc/pam.d/ocserv to use SSSD for authentication. This
is system-specific. A typical Fedora system by default will use SSSD on the host enrolled into
FreeIPA, as shown below.

```
#%PAM-1.0
auth       include	password-auth
account    required	pam_nologin.so
account    include	password-auth
session    include	password-auth
```

Alternatively, to require users to present a certificate for VPN login, the following lines are
required in ocserv.conf. That would require the user to present a signed by IPA certificate in
addition to their password or any Kerberos ticket. That effectively, enables [second factor authentication](ocserv-2fa.md)
with smart cards.

```
auth = certificate
ca-cert = /etc/ipa/ca.crt
crl = /etc/ocserv/MasterCRL.bin
cert-user-oid = 2.5.4.3
```

Since the CRL isn't available as a file in the filesystem you'll need to
download the URL using a periodic cron job from the URL shown below, and then move
the file to the expected location (/etc/ocserv/MasterCRL.bin).

```
wget https://master.ipa.example.com/ipa/crl/MasterCRL.bin
mv MasterCRL.bin /etc/ocserv/
```

Note that, for versions of ocserv prior to 0.10.9 the Master CRL will have to be converted
to PEM format.


### Setting up single sign-on with MS-KKDCP

The MS-KKDCP proxy protocol is a protocol which allows access to the Kerberos
authentication server (KDC) via HTTPS. That enables users to obtain Kerberos tickets
even when outside the local network, allowing them to also login to VPN with their ticket, thus
achieving single sign-on.
For that reason, ocserv provides optionally an MS-KKDCP proxy. To enable it the following
lines in /etc/ocserv/ocserv.conf are required. You’ll need to replace the KERBEROS.REALM
with your realm and the master IPA (KDC) address. 

```
kkdcp = /KdcProxy KERBEROS.REALM tcp@KDC-IP-ADDRESS:88
```


## Setting the client up

This section is identical to the [Kerberos document](ocserv-kerberos.md#setting-the-client-up).


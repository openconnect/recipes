# Integrating [FreeIPA](https://www.freeipa.org) with ocserv

Author: Nikos Mavrogiannopoulos	


## Introduction
FreeIPA is an identity and policy management solution for POSIX based systems.
The OpenConnect VPN server can be integrated with it as it natively supports
[Kerberos](ocserv-kerberos.md). This document provides a simplification over
the Kerberos instructions as it takes advantage of the FreeIPA presence.

## Setting the server up

### Setting up the Kerberos principal

It is needed to add a new Principal into Kerberos to be used by the VPN server.
If for example, your server is 'vpn.example.com' you may use the following instructions.

```
ipa service-add VPN/vpn.example.com
ipa-getkeytab -s vpn.example.com -p VPN/vpn.example.com -k /etc/ocserv/key.tab
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
ipa cert-request --principal VPN/vpn.example.com server.csr
ipa service-show VPN/vpn.example.com --out=/etc/ocserv/server-cert.pem
```

### Setting up OpenConnect server

To enable logins with either PAM-SSSD or Kerberos tickets, it is required to
instruct ocserv to use PAM and  GSSAPI for authentication. That can
be done with the following two lines in /etc/ocserv/ocserv.conf.

```
auth = pam
enable-auth = gssapi[keytab=/etc/ocserv/key.tab,require-local-user-map=true,tgt-freshness-time=360]
```

It is also important to modify /etc/pam.d/ocserv to use SSSD for authentication. This
is system-specific, but in a typical Fedora system it is sufficient to use the defaults,
as shown below.

```
#%PAM-1.0
auth       include	password-auth
account    required	pam_nologin.so
account    include	password-auth
session    include	password-auth
```

### Setting up MS-KKDCP

For users outside the LAN to obtain a Kerberos ticket, the MS-KKDCP proxy protocol
can be used. That proxies Kerberos ticket requests over HTTPS, the same protocol
OpenConnect uses. For that reason, ocserv provides optionally an MS-KKDCP proxy.
To enable it the following lines in /etc/ocserv/ocserv.conf are required.
You’ll need to replace the KERBEROS.REALM with your realm and the master IPA (KDC)
address. 

```
kkdcp = /KdcProxy KERBEROS.REALM tcp@KDC-IP-ADDRESS:88
```


## Setting the client up

This section is identical to the [Kerberos document](ocserv-kerberos.md#setting-the-client-up).


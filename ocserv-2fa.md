# Second factor authentication with ocserv

Author: Nikos Mavrogiannopoulos

## Introduction
ocserv allows for multiple authentication factors per session. This
document discusses the options available for one-time passwords and
smart cards.

## OATH: One-time passwords

It is possible to require each user to enter their password and an additional
one time password. That can be done using PAM as the authentication backend and
pam_oath to provide the seeds. The rest of this text assumes that a working PAM
configuration is in place and pam_oath is installed.

In that case, the following lines should be present in ocserv.conf.

```
auth = "pam"
acct = "pam"
```

Then in addition to the system's authentication you need to add the following
line to /etc/pam.d/ocserv.

```
auth requisite pam_oath.so debug usersfile=/etc/users.oath window=20
```

That configuration, expects in /etc/users.oath the keys for the users
to login. To generate an entry for user 'testuser' use the following
command line.

```
echo "HOTP testuser - $(head -c 16 /dev/urandom |xxd  -c 256 -ps)" >>/etc/users.oath
```

You can print the first 5 passwords for the 'testuser' using the following command (where
KEY is replaced with the key generated above).

```
$ oathtool/oathtool -w 5 KEY
```

The user can then use OTP tools in his mobile like FreeOTP (in android app-store),
or a yubikey as a second factor.

### Yubikey

To store that key in a yubikey to be given to user, use the following command
(requires the yubikey personalization tools).

```
$ ykpersonalize -1 -ooath-hotp -aKEY
```

### FreeOTP

Convert the KEY to base32 (I couldn't figure a one liner for that). Then
use the application 'quearcode' to create a QR code which can be imported
in FreeOTP. The QR code text must be the following (with the obvious parts
being replaced).

```
otpauth://hotp/testuser@example.com?secret=BASE32KEY&issuer=COMPANY&counter=1
```

## PKI: Smart cards

It is possible to use openconnect and ocserv using smart cards as a second factor.
This text will guide the steps required to generate the Public Key Infrastructure
(PKI) to achieve that.

### Generate a certificate authority

Smart cards contain public keys, which in order to be accepted by the server
must be signed with a key the server trusts. That key is called the certificate
authority's key, and the signed public key in smart card is called the certificate.

To generate a CA use the following steps.
```
$ certtool --generate-privkey --outfile ca-key.pem
$ cat << _EOF_ >ca.tmpl
cn = "VPN CA"
organization = "Big Corp"
serial = 1
expiration_days = -1
ca
signing_key
cert_signing_key
crl_signing_key
_EOF_
$ certtool --generate-self-signed --load-privkey ca-key.pem \
        --template ca.tmpl --outfile ca-cert.pem
```

The private key of the CA is now stored in ca-key.pem, and the public key
in ca-cert.pem. The private key of the CA is used to sign client certificates,
and as such it should be unaccessible by anyone, but the manager of the CA.
Ideally it should be stored in a Hardware Security Module or smart card (in that
case you may replace ca-key.pem with the equivalent PKCS #11 URL in the subsequent
steps).

### Sign a client's keys

Suppose we have a smart card and need to generate and sign the client's keys. The
following steps are required. We assume that the PKCS #11 URL of the smart card
is known. If not use "p11tool --list-tokens" to find it out, and replace "pkcs11:" with
the actual URL.

```
$ p11tool --generate-rsa "pkcs11:" --label user-vpn-key --login
$ cat << _EOF_ >client.tmpl
cn = "my-user-name"
organization = "MyCompany"
ou = groupname
expiration_days = 600
signing_key
tls_www_client
_EOF_
$ GNUTLS_PIN=XXXX certtool --generate-certificate --load-privkey "pkcs11:...;object=user-vpn-key;type=private" \
  --load-ca-certificate ca-cert.pem --load-ca-privkey ca-key.pem --template client.tmpl --outfile \
  client-cert.pem
$ p11tool --write --load-certificate client-cert.pem --label user-vpn-key --login "pkcs11:"
```

At this point the smart card contains both the client's certificate and private key.

### Configure ocserv for certificate authentication

The VPN server needs to be configured in order to require certificates signed
by the trusted CA, in addition to a username-password pair. That can be done
by having a configuration that at minimum contains the following configuration options.

```
auth = "pam" #or any other password auth method
auth = "certificate"

ca-cert = /path/to/ca-cert.pem
cert-user-oid = 2.5.4.3
cert-group-oid = 2.5.4.11
```

### Configure openconnect client for certificate authentication

The client can connect to the server by specifying the PKCS #11 URLs of
his certificate and private key (the -c and -k parameters). Note that,
you may specify the minimum URL required, e.g., a URL which identifies the
card and the object name only, and openconnect will expand as necessary.

See also [Smart Card / PKCS#11 support](http://www.infradead.org/openconnect/pkcs11.html)


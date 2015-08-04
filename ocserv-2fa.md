# Second factor authentication with ocserv

Author: Nikos Mavrogiannopoulos

## Introduction
ocserv allows for multiple authentication factors per session. This
document discusses some of the available options.

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


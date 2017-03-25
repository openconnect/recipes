# How to share the same port for VPN and HTTP

Author: Nikos Mavrogiannopoulos

One of the advantages of ocserv is that is an HTTPS-based protocol
and it is often used over 443 to allow bypassing certain firewalls.
However the 443 TCP port is typically used by an HTTP server
on a system. This section will describe two methods on how to collocate
ocserv with a web server.

We recommend the first method, as it has no inherent limitations, as
opposed to the second.

## Method 1: SSL termination on ocserv with haproxy

An alternative method to collocate ocserv and an HTTPS server on port 443,
is by using the server name indication (SNI) present on the first SSL/TLS
ClientHello message, and forwarding traffic according to the name present.

In the example below we assume that the web server and ocserv have to be setup
to use an alternative port, e.g., ocserv uses 4443, and the web server uses
4444. We also assume that the web server responds to www.example.com, while
the vpn server, to vpn.example.com. An example configuration of haproxy that
will redirect the traffic to the appropriate server is shown below.

```
frontend www-https
   bind 0.0.0.0:443
   mode tcp
   tcp-request inspect-delay 5s
   default_backend bk_ssl_default

backend bk_ssl_default
   mode tcp
   acl vpn-app req_ssl_sni -i vpn.example.com
   acl web-app req_ssl_sni -i www.example.com

   use-server server-vpn if vpn-app
   use-server server-web if web-app
   use-server server-vpn if !vpn-app !web-app

   option ssl-hello-chk
   server server-vpn 127.0.0.1:4443 send-proxy-v2
   server server-web 127.0.0.1:4444 check
```

In order for ocserv to obtain information on the incoming session,
we have enabled the proxy protocol in haproxy's configuration (with
the send-proxy-v2 option). That requires ocserv's configuration to contain
the following:

```
listen-proxy-proto = true
```


## Method 1: SSL termination on ocserv with sniproxy

An alternative method to collocate ocserv and an HTTPS server on port 443,
is by using SNI and forwarding traffic accordingly. This example is
identical to the previous, but we use [sniproxy](https://github.com/dlundquist/sniproxy).

Sniproxy allows sharing the HTTPS port as long as the clients advertise
the host name they connect to using server name indication (SNI). This
is true for the majority of web browsers today. For this to work the web
server and ocserv have to be setup to use an alternative port, e.g.,
ocserv uses 4443, and the web server uses 4444. A configuration of sniproxy
that will redirect the traffic to the appropriate server is shown below.

``` 
listener 0.0.0.0:443 {
   protocol tls
   table TableName

   #we set fallback to be ocserv as older versions of openconnect 
   #don't advertise the hostname they connect to.
   fallback 127.0.0.1:4443
}

table TableName {
   # Match exact request hostnames
   vpn.example.com 127.0.0.1:4443
   www.example.com 127.0.0.1:4444
   .*\\.net    127.0.0.1:4444
}
```

Both of the approaches incur a performance penalty and should be considered
mostly for low-traffic VPN servers and web sites.

## Method 2: SSL termination on external program

To collocate ocserv and an HTTPS server on port 443, 
[haproxy](http://www.haproxy.org/) (or similar proxy applications) could
be used. haproxy allows forwarding the HTTPS port data to arbitrary servers,
based on various criteria. This method, however, has few limitations based
on the fact that ocserv does not "see" the SSL session.
 * It cannot enforce client certificate authentication.
 * It cannot derive any keys needed for the DTLS session.
 * It cannot enforce the framing of the SSL/TLS packets, and that
   breaks some assumptions of openconnect client.

Nevertheless, it may be useful on certain scenarios.

The configuration required for haproxy is something along the lines:
```
frontend www-https
    bind 0.0.0.0:443 ssl crt /etc/ocserv/cert-key.pem
    default_backend ocserv-backend

backend ocserv-backend
    server ocserv unix@/var/run/ocserv-conn.socket check
```

and ocserv must be configured to accept cleartext connections on
ocserv-conn.socket file. That can be achieved using the following
configuration snippet.

```
# Accept connections using a socket file. The connections are
# forwarded without SSL/TLS.
listen-clear-file = /var/run/ocserv-conn.socket
```

Note that in that case ocserv will not have enough information
for the client's IP address. That can be fixed by instructing
haproxy to provide that information via the Proxy Protocol (supported
by ocserv 0.10.7 and later). To enable that feature add to ocserv.conf
the following.

```
listen-proxy-proto = true
```

In haproxy.conf you need to enable the following options.
```
backend ocserv-backend
    server ocserv unix@/var/run/ocserv-conn.socket send-proxy-v2 check
```

You could also enable the haproxy's "send-proxy-v2-ssl-cn" option, if
you perform client certificate verification in haproxy and expect
ocserv to trust the user name provided by it.



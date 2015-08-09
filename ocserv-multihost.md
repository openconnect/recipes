# How to share the same port for VPN and HTTP

Author: Nikos Mavrogiannopoulos

One of the advantages of ocserv is that is an HTTPS-based protocol
and it is often used over 443 to allow bypassing certain firewalls.
However the 443 TCP port is typically used by an HTTP server
on a system. This section will describe methods on how to collocate
ocserv with a web server.

## Method 1: SSL termination on external program (haproxy)

To collocate ocserv and an HTTPS server on port 443, 
[haproxy](http://www.haproxy.org/) (or similar proxy applications) could
be used. haproxy allows forwarding the HTTPS port data to arbitrary servers,
based on various criteria. This method, however, has the limitation that
client certificate authentication cannot be enforced by ocserv as
the SSL session is terminated at haproxy.

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
by ocserv 0.10.7 and later). To enable that features add to ocserv.conf
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


## Method 2: SSL termination on ocserv (sniproxy)

An alternative method to collocate ocserv and an HTTPS server on port 443,
is with [sniproxy](https://github.com/dlundquist/sniproxy).
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


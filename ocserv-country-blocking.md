# Country Blocking

Author: Lance LeFlore

When providing VPN services, it is expected that clients will be connecting from a vast number of different subnets around the internet at-large. Because of this, it's not entirely feasible to use a packet filter to whitelist the subnets from which clients may be connecting to the VPN server. Very often, however, one may safely assume that clients will only be connecting from a particular set of countries. This then gives us a parameter we can in turn use to whitelist client connections. For example, if one assumes his or her clients will only ever connect from the US and Czech Republic; a GeoIP lookup can be performed against the clients' source IP to determine whether they are in fact connecting from the US or the Czech Republic.

### Prerequisites

- cURL
- Tested on Ocserv 0.11.5.

### Configuration
- Set ```connect-script = /usr/bin/ocserv-country-blocker``` in ```ocserv.conf```.
- Create a file with the following code at ```/usr/bin/ocserv-country-blocker```:

``` sh
#!/bin/sh

ALLOW_COUNTRY_CODES="CZ US"

# Make sure Ocserv passes IP_REAL
if [ "x${IP_REAL}" = "x" ]; then
  exit 1
fi

GEOIP_STRING=$(/bin/curl -s https://freegeoip.net/csv/$IP_REAL)

COUNTRY_CODE=$(echo $GEOIP_STRING | cut -d ',' -f2)

# Handle instances where COUNTRY_CODE is empty (e.g. RFC 1918 addresses)
if [ "x${COUNTRY_CODE}" = "x" ]; then
  # Fail open
  exit 0
  # Fail closed
  # exit 1
fi

for c in $ALLOW_COUNTRY_CODES; do
  if [ "${c}" = "${COUNTRY_CODE}" ]; then
    exit 0
  fi
done

# If we made it here, then COUNTRY_CODE was not found in ALLOW_COUNTRY_CODES
exit 1
```

- Be certain to set ```ALLOW_COUNTRY_CODES``` to a list of countries you'd like to whitelist VPN access from.
- You should also decide the behavior you'd like the script exhibit in the event that FreeGeoIP does not return a country code. Currently the script "fails open" meaning that it will allow a client connection to proceed if, for some reason, FreeGeoIP does not return a country code.
- Set the executable bit on ```/usr/bin/ocserv-country-blocker```
- Restart Ocserv

### Limitations
- FreeGeoIP enforces an API limit of 10000 API calls per hour per API client source IP.
- The ALLOW_COUNTRY_CODES will obviously affect all VPN clients. If you need a solution to allow clients to connect from different sets of countries then this solution may not suit your use case without some modifications.

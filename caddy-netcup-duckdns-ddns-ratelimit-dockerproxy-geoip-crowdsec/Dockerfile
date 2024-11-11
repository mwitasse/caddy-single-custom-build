# syntax=docker/dockerfile:1
FROM caddy:2.8.4-builder AS builder

RUN xcaddy build \
    --with github.com/caddy-dns/netcup \
    --replace github.com/libdns/netcup=github.com/Monviech/libdns-netcup@libdns-patch \
    --with github.com/caddy-dns/duckdns \
	--with github.com/mholt/caddy-dynamicdns \
	--with github.com/mholt/caddy-ratelimit \
    --with github.com/lucaslorentz/caddy-docker-proxy/v2 \
    --with github.com/hslatman/caddy-crowdsec-bouncer/http \
    --with github.com/porech/caddy-maxmind-geolocation
	
FROM caddy:2.8.4

COPY --from=builder /usr/bin/caddy /usr/bin/caddy

# CMD ["caddy", "docker-proxy"]
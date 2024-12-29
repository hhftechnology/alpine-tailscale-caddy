ARG TAILSCALE_VERSION=v1.78.3

FROM tailscale/tailscale:${TAILSCALE_VERSION}

LABEL maintainer="discourse@hhf.technology" \
      org.opencontainers.image.title="Alpine based Tailscale with Caddy" \
      org.opencontainers.image.description="Alpine-based Tailscale image with Caddy server integration" \
      org.opencontainers.image.version="${TAILSCALE_VERSION}" \
      org.opencontainers.image.vendor="hhf.technology" \
      org.opencontainers.image.url="https://github.com/hhftechnology/alpine-tailscale-caddy" \
      org.opencontainers.image.documentation="https://github.com/hhftechnology/alpine-tailscale-caddy/README.md" \
      org.opencontainers.image.source="https://github.com/hhftechnology/alpine-tailscale-caddy" \
      org.opencontainers.image.licenses="MIT" \
      org.opencontainers.image.created="2024-12-29" \
      com.tailscale.version="${TAILSCALE_VERSION}" \
      com.lika.environment="production"

ENV CADDY_TARGET=
ENV TS_TAILNET=
ENV TS_HOSTNAME=
ENV TS_EXTRA_FLAGS=
ENV TS_USERSPACE=true
ENV TS_STATE_DIR=/var/lib/tailscale/ 
ENV TS_AUTH_ONCE=true

RUN apk update && apk upgrade --no-cache && apk add --no-cache ca-certificates mailcap caddy
RUN caddy upgrade

# Ensure Caddy can access the tailscale socket, Caddy expects it to be under /var/run/tailscale so make a symlink
RUN mkdir --parents /var/run/tailscale && ln -s /tmp/tailscaled.sock /var/run/tailscale/tailscaled.sock

# Add the modified startup script
COPY start.sh /usr/bin/start.sh
RUN  chmod +x /usr/bin/start.sh

# And run it
CMD  [ "start.sh" ]
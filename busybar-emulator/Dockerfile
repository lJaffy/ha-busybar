# BUSY Bar emulator runs on Linux but only binds 127.0.0.1 (server.mjs: HOST
# constant, verified spike). Inside an HA app container 127.0.0.1 is the
# container itself, so LAN clients (BUSY mobile app, busy-hacs on another
# host) could never reach it, and mDNS is absent entirely.
#
# Fix: socat TCP relays, one per port, bound to 0.0.0.0, forwarding into
# 127.0.0.1. IPv4 only (emulator has no IPv6/UDP listeners at all).
#
# Ports relayed:
#   8640 - emulated BUSY Bar HTTP API + /emulator window + state stream WS
#   8099 - ingress proxy target: nginx on 8099 serves the /emulator window
#          through Home Assistant ingress (see ingress.conf)
#
# Emulator sources are fetched in a throwaway stage so the final image
# carries no git client. codeload serves the tag as a tarball; the strip
# leaves packages/, LICENSE and THIRD_PARTY_NOTICES.md at the stage root.
# The layer re-downloads only when EMULATOR_VERSION changes.
ARG EMULATOR_VERSION=v0.1.0-beta.7
ARG BUILD_FROM=ghcr.io/home-assistant/base:3.24
FROM alpine:3.24 AS fetch
ARG EMULATOR_VERSION
RUN apk add --no-cache curl tar \
    && curl -fsSL "https://codeload.github.com/busy-app/busybar-emulator/tar.gz/refs/tags/${EMULATOR_VERSION}" -o /tmp/emulator.tar.gz \
    && mkdir -p /emulator \
    && tar -xzf /tmp/emulator.tar.gz -C /emulator --strip-components=1 \
    && test -f /emulator/packages/engine/build/1.4.1/runtime.wasm \
    && test -f /emulator/packages/engine/build/1.3.1/runtime.wasm \
    && rm /tmp/emulator.tar.gz

ARG BUILD_FROM
FROM ${BUILD_FROM}

ARG BUILD_ARCH
ARG BUILD_VERSION

ENV LANG=C.UTF-8 \
    BUSY_EMULATOR_NO_UPDATE_CHECK=1 \
    EMU_PORT=8640 \
    EMU_DATA=/data/emulator

# Node 22+ is required by the emulator; nginx to serve the window UI through
# ingress with Supervisor-only access. (No socat: busy-relay in
# rootfs/usr/bin rewrites the Host header, which the emulator's entry guard
# requires to be 127.0.0.1:<port> — a plain TCP forward gets a 403.)
RUN apk add --no-cache nodejs nginx python3 \
    && mkdir -p /run/nginx /opt/busy-emulator /data/emulator \
    # Our nginx.conf replaces the stock one (see rootfs/etc/nginx); drop
    # Alpine's default vhost so only ingress.conf (port 8099) is served.
    && rm -f /etc/nginx/nginx.conf /etc/nginx/http.d/default.conf

# Emulator core only: headless server + firmware WASM. Electron app
# (packages/app), 3D stage (packages/stage, proprietary Tride player +
# (c) Flipper 3D model) and dev tools are deliberately excluded.
COPY --from=fetch /emulator/packages/server /opt/busy-emulator/server
COPY --from=fetch /emulator/packages/engine /opt/busy-emulator/engine
COPY --from=fetch /emulator/LICENSE /emulator/THIRD_PARTY_NOTICES.md /opt/busy-emulator/
RUN node --check /opt/busy-emulator/server/index.mjs \
    && test -f /opt/busy-emulator/engine/build/1.4.1/runtime.wasm \
    && test -f /opt/busy-emulator/engine/build/1.3.1/runtime.wasm

COPY rootfs /
COPY ingress.conf /etc/nginx/http.d/ingress.conf

LABEL \
  io.hass.version="${BUILD_VERSION}" \
  io.hass.type="app" \
  io.hass.arch="${BUILD_ARCH}" \
  org.opencontainers.image.source="https://github.com/lJaffy/ha-busybar" \
  org.opencontainers.image.description="Unofficial community packaging of the BUSY Bar emulator (GPL-2.0-or-later) as a Home Assistant app"

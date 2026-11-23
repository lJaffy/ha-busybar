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
ARG BUILD_FROM=ghcr.io/home-assistant/base:3.24
FROM ${BUILD_FROM}

ARG BUILD_ARCH
ARG BUILD_VERSION
ARG EMULATOR_VERSION=v0.1.0-beta.7

ENV LANG=C.UTF-8 \
    BUSY_EMULATOR_NO_UPDATE_CHECK=1 \
    EMU_PORT=8640 \
    EMU_DATA=/data/emulator

# Node 22+ is required by the emulator; nginx to serve the window UI through
# ingress with Supervisor-only access. (No socat: busy-relay in
# rootfs/usr/bin rewrites the Host header, which the emulator's entry guard
# requires to be 127.0.0.1:<port> — a plain TCP forward gets a 403.)
RUN apk add --no-cache nodejs nginx python3 \
    && mkdir -p /run/nginx /opt/busy-emulator /data/emulator

# Emulator core only: headless server + firmware WASM. Electron app
# (packages/app), 3D stage (packages/stage, proprietary Tride player +
# (c) Flipper 3D model) and dev tools are deliberately excluded.
# Shallow-clone the pinned release tag for reproducibility; the Dockerfile
# layer caches it until EMULATOR_VERSION changes.
RUN git clone --depth 1 --branch "${EMULATOR_VERSION}" \
      https://github.com/busy-app/busybar-emulator.git /tmp/emulator \
    && cp -r /tmp/emulator/packages/server /opt/busy-emulator/server \
    && cp -r /tmp/emulator/packages/engine /opt/busy-emulator/engine \
    && cp /tmp/emulator/LICENSE /opt/busy-emulator/ \
    && cp /tmp/emulator/THIRD_PARTY_NOTICES.md /opt/busy-emulator/ \
    && rm -rf /tmp/emulator \
    && node --check /opt/busy-emulator/server/index.mjs \
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

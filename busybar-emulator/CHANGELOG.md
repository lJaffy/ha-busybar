# Changelog

## 0.1.1

- Fix all services failing to start on first boot: init-emulator now shares
  port/firmware with the emulator and relay services via S6
  container_environment (S6 gives each run script a fresh env, so exports
  never arrived); longruns fall back to /run files with a clear error.
- Fix nginx crash loop: run as root (stock Alpine nginx user cannot resolve
  under the AppArmor profile), log to stderr via -e flag (nginx opens its
  compiled-in error log before parsing nginx.conf), temp dirs moved to
  /run/nginx/tmp, profile grants updated to match.

## 0.1.0

- Initial experimental release: headless busybar-emulator server
  (v0.1.0-beta.7) as an HA app, socat LAN relay for the 127.0.0.1-only
  listener, ingress flat window UI, watchdog on /api/status/firmware.

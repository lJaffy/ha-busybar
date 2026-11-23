# Changelog

## 0.1.2

- Root-cause fix for the permission-error chain: delete the hand-written
  apparmor.txt. It caused every denial (getpwnam, error log, temp dirs,
  /run/nginx) while adding no real confinement for this stack. The app now
  runs under Docker's default AppArmor profile, like the official nginx
  add-on. nginx.conf returns to near-stock (user nginx, compiled-in temp
  paths), ingress/run only ensures /run/nginx exists for the pid file.

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

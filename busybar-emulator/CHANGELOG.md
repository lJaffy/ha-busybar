# Changelog

## 0.1.6

- Fix "port in use" on every port: the relay and the emulator cannot share
  one port. Relay-first makes the emulator's someoneListens probe refuse
  the port as taken; emulator-first makes the relay's 0.0.0.0 bind fail
  with EADDRINUSE (upstream take() documents this). Two-port split: the
  emulator takes loopback-only port+1 (never mapped), the relay listens on
  the public port and forwards with Host rewritten to the internal one.
  init-emulator renders ingress.conf from a template with the internal
  port; port option is now validated as 1-65534.

## 0.1.5

- Root cause of the silent exit-1: init-emulator/up was an empty file, so
  the oneshot ran nothing and "succeeded" without writing port/firmware;
  the emulator then failed its guard and exited 1 before logging anything.
  up now points at the run script. Also moved the user bundle from
  s6-rc.d/user to user-bundles.d/user (the old location is deprecated and
  logged a warning on every boot).

## 0.1.4

- Diagnose the silent emulator: log node version, port and firmware before
  exec (the emulator buffers all startup output until ready, so a hang
  printed nothing); map app log_level trace/debug to the emulator's
  --verbose flag (prints where a failed start failed).
- Fix emulator/finish: the old s6-svc -O . never took the container down on
  a crash, leaving a dead emulator with zero log output. Crash or
  non-SIGTERM signal now halts the container so the watchdog restarts it.

## 0.1.2

- Root-cause fix for the permission-error chain: delete the hand-written
  apparmor.txt. It caused every denial (getpwnam, error log, temp dirs,
  /run/nginx) while adding no real confinement for this stack. The app now
  runs under Docker's default AppArmor profile, like the official nginx
  add-on. nginx.conf returns to near-stock (user nginx, compiled-in temp
  paths), ingress/run only ensures /run/nginx exists for the pid file.
- Fix relay collapsing under burst load (Errno 24, no file descriptors):
  bounded 100-connection worker pool with socket timeouts and prompt close
  instead of unbounded thread-per-connection; relay-api now exits non-zero
  if the emulator never comes up, so s6 restarts instead of proxying into
  a refused port.

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

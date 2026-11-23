# BUSY Bar Emulator (unofficial community app)

> **Unofficial packaging.** This app is maintained by
> [lJaffy](https://github.com/lJaffy/ha-busybar) and is not affiliated with
> or endorsed by BUSY. It bundles the open-source
> [busybar-emulator](https://github.com/busy-app/busybar-emulator) (GPL-2.0-or-later)
> as a Home Assistant app.

A self-contained virtual [BUSY Bar](https://busy.app): the open-source
[busybar-emulator](https://github.com/busy-app/busybar-emulator) headless
server (firmware 1.4.1 or 1.3.1, API on port 8640) plus its flat window UI
through the Home Assistant sidebar.

Control it with the [BUSY HACS integration](https://github.com/busy-app/busy-hacs):
sessions, notifications, drawings, buttons, wheel, camera entity — exactly like
a physical bar. Pair the BUSY mobile app with it to lock down your phone and
focus.

## Setup

1. Install and start this app.
2. In the HACS **BUSY Bar** integration, add a bar manually:
   host `http://<home-assistant-ip>:8640`.
   - When asked for a Wi-Fi access key: the emulator runs with an open API
     (no key). Minting a token needs no key — leave any key field empty; if
     the flow insists, enter anything, minting ignores it.
   - The entry stores a token shown on the virtual bar as `HA <name>`.
3. Optional: enable **Show in sidebar** on this app's page for the `/emulator`
   window (front/back displays, controls) inside Home Assistant.

## How it works

- The emulator server binds `127.0.0.1` only (upstream design). A `socat`
  relay exposes it on the container network and, via `ports`, on your LAN as
  `<ha-ip>:8640`. Same port number inside and out.
- Firmware choice (`latest` = 1.4.1, or 1.3.1) and the virtual bar's installed
  JS apps live in `/data/emulator` — persistent across restarts and included
  in backups.
- Ingress serves the flat window UI. The 3D view is excluded (proprietary
  player + model, desktop-GPU oriented).

## Limitations (verified against emulator v0.1.0-beta.7)

- **No mDNS advertisement.** The emulator has no UDP/mDNS listener, so the
  integration's zeroconf discovery won't find it — add it by address.
  The integration follows address changes once configured.
- **No Wi-Fi setup screens.** The emulated bar reports a stub Wi-Fi
  (`Emulator` SSID, `192.168.1.100`); configure pairing by URL instead.
- **Fresh-out-of-box settings each start** (volume, name, timezone) — upstream
  emulator behaviour. Installed apps persist; bar settings don't (yet: a
  `backup_pre` snapshot of settings is a possible improvement).
- **BUSY mobile app pairing is untested.** If the app requires BUSY cloud
  account linking or BLE, use the fallback: Home Assistant Companion App
  automations (Do-Not-Disturb, lock) triggered by the virtual bar's session
  sensors.
- **The update check is disabled** (`BUSY_EMULATOR_NO_UPDATE_CHECK=1`).
  Update by updating this app.
- aarch64 images are published but the firmware WASM is verified on x86-64
  first; report Pi issues upstream.

## Options

| Option | Default | Description |
|--------|---------|-------------|
| `port` | `8640` | Host + container port of the emulated bar API |
| `firmware` | `latest` | `latest` (= 1.4.1), `1.4.1`, or `1.3.1` |
| `log_level` | `info` | Supervisor log level |

## License

Emulator: GPL-2.0-or-later © Flipper FZCO, see `/opt/busy-emulator/LICENSE`
in the image and `THIRD_PARTY_NOTICES.md`. Tride 3D player and bar 3D model
are excluded from this package.

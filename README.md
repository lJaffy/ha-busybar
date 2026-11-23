# BUSY Bar apps for Home Assistant (unofficial)

> **Community packaging, not an official BUSY project.** Maintained at
> [lJaffy/ha-busybar](https://github.com/lJaffy/ha-busybar). Upstream
> emulator and integration links below belong to BUSY.

Self-contained [BUSY Bar emulator](https://github.com/busy-app/busybar-emulator)
packaged as a Home Assistant app (formerly add-on): a virtual BUSY Bar with its
HTTP API and flat window UI, controllable by the
[BUSY HACS integration](https://github.com/busy-app/busy-hacs) — no physical
device needed.

## Apps

| App | Description |
|-----|-------------|
| `busybar-emulator` | Virtual BUSY Bar: HTTP API on port 8640, window UI via ingress/sidebar |

## Install

1. Settings → Apps → Install app → ⋮ → Repositories → add this repo URL.
2. Install **BUSY Bar Emulator**, start it.
3. Add the HACS integration **BUSY Bar** (custom repository
   `https://github.com/busy-app/busy-hacs`), then add the virtual bar:
   manual host `http://<home-assistant-ip>:8640` (auto-discovery via mDNS is
   not available in-container yet — see `busybar-emulator/DOCS.md`).
4. Pair the BUSY mobile app with `http://<home-assistant-ip>:8640` for phone
   lockdown / focus sessions driven by the virtual bar.

## Status

Experimental. amd64 + aarch64. See each app's `DOCS.md`.

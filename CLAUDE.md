# CLAUDE.md — water-mesh-config

Guide for Claude Code agents working in this repo.

This is the **Waterbear Field School** fork of edgecollective's `micro-config` project. It is deployed as a GitHub Pages project site under the `waterbearfieldschool` org and served at **`waterbearfieldschool.org/water-mesh-config/`** (the org's custom domain applies to project sites, so no per-repo `CNAME` is needed). It's linked from the WFS site's journal + Resources section.

## What this is

A fork of the [MeshCore flasher](https://flasher.meshcore.io) web UI, tailored into a **guided setup page for a specific two-device kit**: an ultrasonic water-level sensor (Rook v4 as sender + Heltec V3 as receiver) that bridges LoRa DMs to Bayou over WiFi. Note: the sender is **Rook v4 hardware**, but its firmware is versioned separately — the current build is tagged `v3-ultrasonic` (files named `rook_sender_ultrasonic_v3.uf2`). Don't conflate the hardware rev with the firmware version. The upstream project is a generic multi-device flasher; this fork narrows it to one hardware pair, adds an explainer / TOC / help-menu sections, and hosts the firmware bins locally.

Deploy target: a GitHub Pages project site at `waterbearfieldschool.org/water-mesh-config/`. Users open it in Chrome/Edge, flash the Heltec V3 directly over Web Serial, drag the Rook UF2 onto the NICENANO drive, then use the in-page serial console to configure WiFi + pair the devices.

## Architecture

Vue 3 SPA, single `index.html` + `flasher.js` + `simple-sensor.json`. **All page copy — section titles, intros, help output, common-settings bullets — lives in `simple-sensor.json`.** The HTML template is a thin renderer that pulls from that JSON via `v-html` / `{{ }}`. To edit user-visible text, edit the JSON, not the HTML.

- `index.html` — template. Sections rendered in order: header logos + title → TOC → §1 Overview → §2 Flashing → §3 Connecting via Advert → §4 Configuration.
- `flasher.js` — Vue setup. Loads `?config=<name>.json` (default `simple-sensor`), fetches SVGs and inlines them (see below), extends `commandReference` from `config.commands`.
- `simple-sensor.json` — the source of truth for all copy and device definitions. Devices, firmware file references, section text, TOC labels, help output — all here.
- `firmware/simple-sensor/*.{bin,uf2}` — bundled snapshots. Rename them with a version suffix (e.g. `_v3`, `_v4`) each release and update `simple-sensor.json` to match.
- `img/sensor-system.svg` + `img/sensor-system-mobile.svg` — hero graphic. Both inline PNG references to `img/*.png`; must be **inlined** to render (see next).
- `css/flasher.css` — layout, sections, mobile breakpoints (`@media (max-width: 640px)` and `380px`).

## Key patterns

**Inline SVG.** When an SVG is loaded via `<img src=...>`, browsers block internal external references (`<image href=...>`) for security. So `flasher.js` fetches the SVG file as text and injects it via `v-html`. The desktop and mobile SVGs both use this path — see `explainerSvgInline` / `explainerSvgMobileInline`.

**Mobile-friendly SVG.** Two separate SVG files (horizontal for desktop, vertical stack for mobile), toggled by `.svg-desktop` / `.svg-mobile` CSS at 640px.

**Beer.css nav gotcha.** Beer.css styles `<nav>` as an inline-flex button row. Wrap content lists (e.g. the TOC) in `<div>`, not `<nav>`, or the inner `<ol>` collapses invisibly.

**Config JSON conventions.** Section keys: `explainer`, `flashingSection`, `contactExchange`, `configuration` (with `intro` + `commonSettings` + `helpText`). Per-device: `firmware[].version['<version-tag>'].files[]` with `type` in `download`/`flash-wipe`/`flash-update`.

**Subpath portability.** All asset paths are relative (no leading `/`) so the site works when hosted at any subpath (e.g. `waterbearfieldschool.org/water-mesh-config/`). The SPA URL routing is fragment-based (`#/rook-v4/...`) rather than pathname-based, so navigation state changes don't push the browser out of the deployment subpath and no server rewrites are needed. When adding new asset paths anywhere (HTML, JSON, JS, SVG), keep them relative.

## Running locally

```
python3 -m http.server 8765
# open http://localhost:8765/
```

`http.server` binds to all interfaces (`0.0.0.0`) by default, so a phone on the same WiFi can reach `http://<LAN-IP>:8765` — find the IP with `hostname -I`. Add `--bind 127.0.0.1` only if you want to restrict access to localhost. For cellular / real testing, use `cloudflared tunnel --url http://localhost:8765`.

## Web Serial caveats

The serial console + web flash use the Web Serial API. It requires HTTPS (or localhost) and works only in Chrome/Edge on desktop — Firefox, Safari, and all mobile browsers can view the page but cannot open the console. Once Chrome grabs a serial port, it holds it exclusively — that's the recurring source of "cannot open /dev/ttyUSB0" errors during firmware development.

## Version bumping firmware

1. Build new bins in the parent `MeshCore-simple-sensor` repo.
2. Copy into `firmware/simple-sensor/` with a bumped version suffix (`_v3` → `_v4` etc.).
3. Update the file `name` and `title` fields in `simple-sensor.json` under the appropriate `firmware[].version[].files[]`.
4. Also bump the `version` key itself if you want to communicate the change to the user.

## What NOT to change without asking

- Upstream `config.json` — kept alongside `simple-sensor.json` so `?config=config` still works for the original flasher use case.
- `lib/`, `flasher.js` core flashing logic (`flashDevice`, `Dfu`, `ESPLoader` usage) — inherited from upstream. Only patch when necessary.

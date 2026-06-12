<p align="center">
  <img src="https://brands.home-assistant.io/locale/logo.png"
       alt="Locale Systems" height="84">
</p>

# Locale — Home Assistant

Home Assistant support for Locale Systems devices — pool controllers,
salt-water chlorinators, and related equipment. Communicates entirely
over your local network with real-time push updates. No cloud
dependency.

It comes as **two pieces from this one repository**:

- the **Locale add-on** — discovers your devices on the network,
  holds Home Assistant's device credential, serves devices time, and
  receives their telemetry;
- the **Locale integration** (HACS) — presents the devices the add-on
  manages as native Home Assistant entities.

## Requirements

- Home Assistant 2025.10 or later.
- A Locale device on current firmware.
- The **Locale Home** iOS app (the device's admin authorizes Home
  Assistant from there).
- The add-on host and the devices on the same local network segment
  (mDNS + time service).

## Installation

Use this repository's URL in **both** stores:

**Add-on** (Home Assistant OS / Supervised):

1. **Settings → Add-ons → Add-on Store → ⋮ → Repositories**, add
   `https://github.com/Locale-Systems/locale-homeassistant`.
2. Install **Locale** and start it.
3. Copy the API token from the add-on log.

(Running HA Core/Container without a Supervisor? Run the add-on image
with Docker instead — `ghcr.io/locale-systems/locale-ha-addon`.)

**Integration** (HACS):

1. In HACS, add the same URL as a custom repository (type:
   Integration).
2. Search for **Locale**, install, and restart Home Assistant.

## Setup & onboarding

1. **Add Integration → Locale**, enter the add-on host/port (defaults
   are right for an add-on on the same machine) and the API token.
2. Open the integration's **Configure** menu → **Adopt devices**.
   Home Assistant shows a short *request* (QR or text) to hand to
   your device admin. In the **Locale Home** iOS app, the admin
   reviews it and hands back a *grant* you paste into Home Assistant.
3. Adopted devices appear automatically within about half a minute —
   every authorized Locale device shows up under the single Locale
   entry. Run **Adopt devices** again anytime to add more.

## What you get

Entities are created automatically from what the device reports —
sensors, switches, numbers, selects, and a climate control where the
device exposes one. Each device also gets:

- **Diagnostic attributes** — probe status, calibration dates,
  confidence, demand vs. actual, etc., attached to the relevant
  entity instead of spawning extra ones.
- **Fault sensors** — a companion fault binary sensor per entity,
  carrying the device's fault code and message when active.
- A **Connection** diagnostic sensor reflecting the device's
  reachability on your network.

### Firmware updates

When the integration is configured with your platform's firmware
service, each device gets:

- a **Firmware** update entity showing the installed and latest
  available version,
- a **Firmware version** selector to pick a specific published build,
- **Check for firmware updates** and **Install selected firmware**
  buttons.

Installing streams the image straight to the device and it reboots
automatically; the device page reflects the new version once it's
back.

## Offline behavior

The integration caches what it needs so devices still render (as
unavailable) while reconnecting. Devices with built-in schedules
resume them automatically whenever Home Assistant isn't connected.

## Support

Issues and feature requests:
<https://github.com/Locale-Systems/locale-homeassistant/issues>

Artwork is served by Home Assistant from the community
[brands](https://github.com/home-assistant/brands) service; until it
is published there the integration uses Home Assistant's generic icon
(functionality is unaffected).

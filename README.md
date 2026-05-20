<p align="center">
  <img src="https://brands.home-assistant.io/locale/logo.png"
       alt="Locale Systems" height="84">
</p>

# Locale — Home Assistant Integration

Home Assistant integration for Locale Systems devices — pool
controllers, salt-water chlorinators, and related equipment.
Communicates entirely over your local network with real-time push
updates. No cloud dependency.

## Requirements

- Home Assistant 2025.10 or later.
- A Locale device on current firmware.
- The **Locale Home** iOS app (the device's admin authorizes Home
  Assistant from there).
- For automatic discovery, Home Assistant and the device should be on
  the same local network.

## Installation (HACS)

1. In HACS, add `https://github.com/Locale-Systems/locale-homeassistant`
   as a custom repository (type: Integration).
2. Search for **Locale**, install, and restart Home Assistant.

## Onboarding

1. After restart, a **Locale** device usually appears under
   **Settings → Devices & Services** within a minute (or add it
   manually with **Add Integration → Locale**).
2. Home Assistant shows a short *request* to hand to your device
   admin. In the **Locale Home** iOS app, the admin reviews it and
   either delivers the authorization directly over the network or
   hands back a *grant* you paste into Home Assistant.
3. One Home Assistant device is created per authorized Locale device.

If the device's authorization is later revoked or reset, Home
Assistant raises a repair you can fix the same way.

## What you get

Entities are created automatically from what the device reports —
sensors, switches, numbers, selects, and a climate control where the
device exposes one. Each device also gets:

- **Diagnostic attributes** — probe status, calibration dates,
  confidence, demand vs. actual, etc., attached to the relevant
  entity instead of spawning extra ones.
- **Fault sensors** — a companion fault binary sensor per entity,
  carrying the device's fault code and message when active.
- A **Connection** diagnostic sensor for the live-update stream
  health.

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

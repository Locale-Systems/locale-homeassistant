# Locale Add-on

The Locale add-on is the hub between your Locale devices (pool
controllers, chlorinators — anything built on locale-core) and Home
Assistant. It discovers devices on the LAN, holds Home Assistant's
device credential, serves devices time, receives their telemetry, and
exposes everything to the **Locale integration** (installed separately
via HACS) over a local API.

The integration never talks to devices directly — install both pieces.

## Installation

1. Add this repository to the add-on store (it's the same URL as the
   HACS repository).
2. Install **Locale** and start it.
3. Copy the API token from the add-on log (first start prints it;
   it's also persisted in the add-on's data directory).
4. Install the **Locale** integration via HACS and configure it with
   this host, port `8099`, and that token.
5. In the integration's **Configure** menu, run **Adopt devices** and
   complete the grant exchange with the Locale Home iOS app.

## Network

The add-on uses `host_network` — it needs L2 reach for mDNS device
discovery and to serve SNTP (UDP 1123) to devices. Ports:

| Port | Purpose |
|------|---------|
| 8099/tcp | Local API for the integration |
| 1123/udp | SNTP time service for devices |
| 8443/tcp | Telemetry receiver (HTTPS, Root-validated) |

## Options

| Option | Description |
|--------|-------------|
| `log_level` | `debug` / `info` / `warn` / `error` |

## Support

Issues: https://github.com/Locale-Systems/locale-homeassistant/issues

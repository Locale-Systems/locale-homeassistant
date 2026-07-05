# Locale Add-on

The Locale add-on is the always-on **hub** between your Locale devices
(pool controllers, chlorinators — anything built on locale-core) and Home
Assistant. It discovers devices on the LAN, holds Home Assistant's signing
identity, serves devices time, receives their telemetry, optionally bridges
telemetry to the Locale cloud, and exposes everything to the thin **Locale
integration** (installed separately via HACS) over a local API.

The integration is presentation only — it talks **exclusively** to this
add-on and never contacts devices directly. Install both pieces.

The one authority in the system is your phone: the **Locale Home iOS app**
holds the installation Root key. The add-on is just another principal the
phone enrolls — like a device. All management (adopting devices, rotating
keys, removing grants) lives on the phone; the add-on serves only a minimal,
setup-only pairing page, not a management console.

---

## Installation

The add-on ships the same prebuilt image two ways:

- **Home Assistant OS / Supervised:** add this repository to the Supervisor
  add-on store, install **Locale**, and start it. Supervisor pulls the
  prebuilt image and maps `/data` for you.
- **HA Core / Container (or any Docker host):** run it alongside Home
  Assistant with `docker compose -f deploy/docker-compose.yaml up -d`.

Then install the **Locale** integration via HACS. In most setups you will
not type a host, port, or token — see [Connecting the integration](#connecting-the-integration).

**Networking matters.** The add-on uses host networking so it has L2 reach
for mDNS discovery, for serving SNTP, and for same-subnet pairing. On a
Docker bridge (common on macOS/Windows) mDNS and same-subnet detection both
fail together; the manual fallback still works but you lose the zero-paste
paths. A host-networked install (the HAOS add-on, or the recommended compose
file) is strongly preferred.

---

## Pairing the add-on to your phone (enrollment)

Before the add-on can adopt any devices it must be **enrolled** by your
phone's Root key. This is a one-time ceremony that establishes the add-on's
identity and installs its certificates. It substitutes for the physical
BLE-presence proof a device uses, reconstructing "presence" from three terms
that must all hold: a deliberate arm action, being on the same subnet, and a
short-passcode PAKE.

1. **Open the setup page.** Under Supervisor, open the **Locale Pairing**
   panel in the HA sidebar (served over ingress, so HA-admin auth is free —
   no host or port to type). Standalone, browse to the add-on host directly
   (e.g. `http://locale.local/`, advertised over mDNS) on the setup port.
2. **Arm pairing.** Click **Arm pairing**. The add-on generates a short,
   single-use, time-boxed passcode (~5 min) and renders a **QR code** plus
   the code. The QR carries the add-on's mDNS instance id, LAN endpoint
   hints, its self-generated Issuer public-key fingerprint, and the passcode.
3. **Scan with the Locale Home app.** The iOS app scans the QR (or you type
   the code), finds the add-on on the LAN, and runs a **SPAKE2+ PAKE** over
   the passcode. The PAKE turns the un-anchored first contact into a
   mutually-authenticated channel and binds *passcode ↔ add-on pubkey ↔
   phone* — this is what proves "this is **my** add-on, not a rogue on the
   subnet." Same-subnet is also enforced server-side as defense-in-depth.
4. **The phone mints the grant.** Over that channel the Root mints one grant
   **batch** and seals it to the add-on's now-verified key. Applying it
   installs the add-on's whole identity (see
   [Identity & certificates](#identity--certificates)) and adopts the devices
   you selected.
5. **Enrolled.** The mobile↔add-on mTLS edge goes live, the integration's
   bearer is issued locally, and the page shows success. From here on the
   phone manages everything over mTLS; the pairing window disarms.

**Bridged / NAT fallback.** When mDNS and same-subnet detection fail (Docker
bridge, segmented VLANs), the `/pair` page still renders the QR/code but you
enter the add-on's host manually in the app and deliver the grant by paste.
Same result, two more steps.

---

## Identity & certificates

The add-on generates two independent, **firewalled** keypairs on first start
and persists them under its data directory. Enrollment layers Root-signed
certificates on top of the first one.

**Device-plane Issuer key (`role:ha`).** A P-256 keypair generated at first
start — the add-on's signing identity (this is the key the integration's
`auth.py` held before the two-piece split). One Issuer key covers every
device this add-on onboards. During enrollment the phone's Root mints, and
the add-on installs:

- a Root-signed **`role:ha` Issuer cert** — the add-on acting as itself
  (residual control, time-sync);
- a Root-signed **`role:relay` Issuer cert** — the add-on acting as a conduit
  that forwards an owner's phone request to a device;
- the installation **Root SPKI** and, from the Issuer cert, the
  `installation_id` + `root_epoch`. Together these form the **installation
  anchor** the add-on uses to validate a connecting phone's client cert.

From the two Issuer certs the add-on builds **mTLS carrier leaves** (one per
role) that it presents on the **addon→device** edge. Device trust keys to the
embedded, Root-signed Issuer cert — not the leaf signature — so the device
authenticates the connection against the Root it already anchors. The device
edge is **mandatory mTLS**; the old per-request bearer path is retired on
both device and add-on.

**Server TLS leaf.** The telemetry hub and the mobile mTLS edge are served
with a certificate keyed to that same `role:ha` key. It is **self-signed**
until enrollment delivers a **Root-signed X.509 leaf**, which then replaces it
live (no restart). Devices validate it against the installation Root anchor
they already hold — no per-peer pinning.

**Cloud-plane identity.** A **separate** P-256 keypair, kept distinct from the
Issuer key so cloud auth and device authz never cross. Its public half is what
you register as the Home's **HA service credential** (via the iOS Root) to
enable the cloud tier; the add-on signs platform challenges with the private
half to mint short-lived home-tokens. Nothing forwards to the cloud until this
credential is registered.

**Integration bearer token.** Distinct from all of the above — it is a local
API key for the HACS integration only (see below), not part of the crypto
identity. It grants the **operational** API surface only; grant-changing
operations (re-pair, renew key, remove grant) always require the Root via the
iOS app.

---

## Connecting the integration

Getting the integration talking to the add-on is a **separate, local**
concern from phone enrollment, and in the common cases involves no typing:

- **HAOS / Supervised:** the add-on publishes a Supervisor **discovery**
  message over the private Supervisor channel; the integration is
  offered/auto-configured with host, ingress, and bearer — nothing pasted.
- **HA Core / Container:** the integration finds host/port via **zeroconf**
  (`_locale-addon._tcp`) and obtains its bearer from the co-resident add-on
  via a local handshake gated by the arm action / local origin. The `/pair`
  page can also reveal/copy the bearer.
- **Manual fallback (bridged networks):** enter the host, port `8099`, and
  the bearer token by hand. The token is persisted at `<data>/api_token` and
  its location (never the value) is printed in the add-on log on first start.

Once connected, use the integration's **Configure → Adopt devices** menu to
run the grant exchange described above with the iOS app.

---

## Cloud tier (optional)

The add-on works fully offline — it is an always-on local hub with no cloud
dependency. Two optional cloud behaviors light up only when you connect the
Home to the Locale cloud from the iOS app:

- **Firmware / OTA:** if `platform_url` is set (it defaults to the Locale
  cloud), the add-on reads the firmware registry and can offer OTA. It mints
  its own device-scoped provision token — no platform credential is stored in
  the add-on or the integration. Leave `platform_url` blank to disable.
- **Telemetry forwarding:** with `telemetry_forward` on **and** an HA service
  credential registered, the add-on drains its local telemetry store to the
  cloud under a home-token (the HA-bridged tier). With no credential nothing
  forwards regardless, so this is opt-in by connecting the Home. Set
  `telemetry_forward` false to keep telemetry local even after connecting.

---

## Network

Host networking is required. Ports:

| Port | Proto | Purpose |
|------|-------|---------|
| 8088/tcp | HTTP | Setup / pairing UI (also via Supervisor ingress) |
| 8099/tcp | HTTP | Local API for the integration (bearer-gated) |
| 8076/tcp | HTTP | LAN-direct grant receiver (arm-gated; ECIES-sealed) |
| 8100/tcp | HTTPS | Mobile→add-on mTLS API (owner client cert) |
| 8443/tcp | HTTPS | Telemetry receiver (Root-validated) |
| 1123/udp | SNTP | Device time service (Internet-Disabled tier) |

The setup UI, onboard receiver, and telemetry/mobile ports are also
advertised over mDNS (`_locale-onboard._tcp` while armed;
`_locale-addon._tcp` and `_locale-relay._tcp` persistently) so phones and the
integration find the add-on without a typed address.

---

## Options

| Option | Default | Description |
|--------|---------|-------------|
| `log_level` | `info` | `debug` / `info` / `warn` / `error`. |
| `sntp_advertise` | *(blank)* | `host:port` to hand adopted devices as their NTP server, e.g. `192.168.1.10:1123` — the HA host's LAN address (the device dials it). Blank = leave device NTP untouched. |
| `telemetry_advertise` | *(blank)* | `https://host:port` to point adopted devices' telemetry at this add-on, e.g. `https://192.168.1.10:8443`. Blank = devices post direct. |
| `platform_url` | Locale cloud | Cloud base URL for firmware/OTA. Override for dev/self-host; blank disables OTA. |
| `telemetry_forward` | `true` | Forward device telemetry to the cloud under a home-token (only takes effect once the Home is cloud-connected). Set false to keep telemetry local. |

`sntp_advertise` / `telemetry_advertise` are site-specific and can't be
defaulted — the add-on logs the recommended values for your network at
startup, ready to copy into the Configuration tab.

---

## Data directory

The add-on persists everything under its data dir (`/data` under Supervisor;
a named volume in compose). Notable files:

| File | Contents |
|------|----------|
| `api_token` | integration bearer token |
| `ha_issuer_key.pem` | device-plane `role:ha` Issuer keypair |
| `cloud_key.pem` | cloud-plane identity keypair (firewalled) |
| `ha_root_spki.der` | installation Root SPKI (from the grant) |
| `ha_issuer_cert.jws`, `relay_issuer_cert.jws` | Root-signed Issuer certs |
| `ha_server_cert.der` | Root-signed server TLS leaf |
| `adopted_devices.json` | adopted-device store |
| `telemetry/` | local telemetry store + forward queue |

Wiping the data dir resets the add-on to an un-enrolled state — you would
re-pair from scratch.

---

## Support

Issues: https://github.com/Locale-Systems/locale-homeassistant/issues

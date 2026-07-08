# Changelog

## 0.5.11

- feat(cloudtunnel): relay config-write + reboot over the tunnel
- chore: pin locale-go-common to merged mgmt-grammar rev

## 0.5.10

- addon: use go-common VerifyRaw / ParseP256SPKI / DecodePayload
- addon: shared relay refusal rendering + target-device header constant

## 0.5.9

- addon: whole-home multiplexed tunnel /events (home-events-multiplexing step 1)
- addon: tunnel dispatch reads device_id from AddonService body

## 0.5.8

- refactor(relay): use shared gorelay.EventsForwardDecision

## 0.5.7

- feat(cloudtunnel): stream device /events over the add-on tunnel (Phase 2)

## 0.5.6

- cloudtunnel: use shared tunnelwire frame codec

## 0.5.5

- api: connect-cloud over the mobile mTLS edge (no QR)

## 0.5.4

- cloudtunnel: quiet idle when no HA credential + dispatch log

## 0.5.3

- cloudtunnel: add-on WS tunnel to platform (platformsse Phase 1, add-on side)

## 0.5.2

- refactor(relay): collapse prepare/prepareProven onto shared ForwardDecision
- chore(deps): repin locale-go-common (ForwardDecision)

## 0.5.1

- fix(devices): cap management transport at one connection per device
- feat(relay): forward OTA as the relayed owner (PROVEN ota)

## 0.5.0

- Revise addon documentation to reflect current state
- chore(deps): repin locale-contracts + locale-go-common (role:agent)
- feat(relay): carry subject_token + forward PROVEN management ops

## 0.4.3

- go.mod: bump contracts pin to cc6b63a
- ci(release): generate store CHANGELOG.md from tags

## 0.4.2

- Add standalone local-handshake bearer release; stop logging API token

## 0.4.1

- release: generate contracts authz constants (go-common dep)

## 0.4.0

- Add SPAKE2+ pairing: PAKE engine, setup UI, responder
- Add integration auto-discovery (zeroconf + Supervisor)
- ci: generate contracts authz constants (go-common dep)

## 0.3.12

- refactor(mobileauth,api): consume go-common observable edge
- build: bump locale-go-common pin to the merged observable-edge commit

## 0.3.11

- fix(devices): canonicalize UUID in manager device lookups

## 0.3.10

- api: structured Twirp errors for relay/device failures (caller no longer blind)

## 0.3.9

- devices/relay: log the relay->device forward outcome
- addon: force HTTP/1.1 on the mobile mTLS edge (test the h2 -1206 hypothesis)

## 0.3.8

- mobileauth: full handshake observability (ClientHello, full fp, http errors)
- mobileauth: add negotiated ALPN + per-request path/source-IP trace

## 0.3.7

- mobileauth: request (not hard-require) the client cert so no-cert is observable

## 0.3.6

- mobileauth: log mTLS handshake diagnostics (why a client cert is rejected)

## 0.3.5

- mobileauth: cap the mobile->addon mTLS edge at TLS 1.2

## 0.3.4

- go.mod: bump go-common pin to the v7-cascade master (85c77f2)
- relaymdns/onboardlan: advertise an explicit LAN A record (fix unresolvable SRV)

## 0.3.3

- bump contracts pin to relay-seam Phase 0 (dfddd38)
- addon: LAN relay + fan-out (relay-seam Phase 2)
- go.mod: bump contracts to corpus v7 and go-common to the merged relay rev

## 0.3.2

- devices: retire the per-request Bearer (mandatory mTLS)

## 0.3.1

_No changes recorded._

## 0.3.0

- addon: Root-anchored mTLS on both LAN edges via locale-go-common
- ci: check out locale-go-common sibling for build + image

## 0.2.2

- devices: tolerate missed mDNS cycles before evicting a live session

## 0.2.1

- logging: route hashicorp/mdns chatter through slog at Debug/Warn
- devices: classify and log SSE termination reasons

## 0.2.0

- config: default platform_url to the Locale cloud
- config: surface recommended sntp/telemetry advertise values
- onboarding: parse haCredentialId from the grant batch (HA-bridged tier A)
- onboarding: Go ONBOARD ECIES + surface haCredentialId (cloud-enrollment plumbing)
- onboardlan: LAN-direct grant receiver (mDNS + sealed /grant)
- cloud: add-on home-token client (HA-bridged tier A, auth only)
- onboarding: emit cloudPubkey in the request blob (HA-bridged tier A)
- telemetry: cloud forwarder under home-token (HA-bridged tier piece B)

## 0.1.12

- firmware: forward per-build publish time to the integration

## 0.1.11

- ota: size install from registry file_size, not download Content-Length

## 0.1.10

- firmware: own the OTA path in the add-on (provision-token auth)

## 0.1.9

- provision: set ntp + telemetry in one /appconfig write (single reboot)

## 0.1.8

- diag: log read-back values when telemetry host needs reprovisioning

## 0.1.7

- fix: graceful shutdown hangs on stop/restart (SSE keeps server active)
- feat: expose sntp_advertise + telemetry_advertise as add-on options

## 0.1.6

- fix: canonicalize device UUIDs (lowercase) + conformance-pin grant parsing

## 0.1.5

- fix: actually apply log_level (slog level + Supervisor options.json)

## 0.1.4

- observability: log onboarding request/grant + reconcile summary
- observability: log every mDNS entry + drop reason (discovery diagnosis)
- fix: query mDNS per-interface so HAOS host_network add-ons find devices

## 0.1.3

- ci: stop publishing images on master; release.yml owns GHCR
- ci: image build-check only when image mechanics change

## 0.1.2

- fix: pairing token is always the persisted one, never SUPERVISOR_TOKEN

## 0.1.1

- fix: run as root so Supervisor's bind-mounted /data is writable

## 0.1.0

- Scaffold locale-ha-addon — Go skeleton + dual distribution
- internal/sntp: SNTP server reflecting the host clock
- internal/mdns: LAN device discovery (hashicorp/mdns)
- internal/identity: HA Issuer keypair + request-token mint
- internal/devices: device client (pinned TLS + Twirp) + contracts build wiring
- internal/devices: grant store + onboarding blob codec
- internal/devices: SSE entity-event stream client
- internal/devices: orchestration Manager (discovery -> live sessions + events)
- api: haaddon AddonService + AddonEvent SSE; wire the manager into main
- api: HTTP onboarding endpoints (request/grant -> adopt)
- #2: sntp device-provisioning — point adopted devices at the add-on
- relay 3a-3c: offline telemetry hub (TLS server + receive + local store)
- relay: serve the Root-signed telemetry-hub leaf from onboarding
- relay: auto-provision the device telemetry host at adoption
- conformance tests: pin the JWS mint + envelope-sig verify against the locale-crypto corpus
- api: device GetVersion passthrough + OTA firmware proxy
- release: publish add-on store artifacts on v* tags

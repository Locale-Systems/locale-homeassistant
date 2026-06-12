# Changelog

## 0.1.0

Initial add-on store release.

- LAN device discovery (mDNS), adoption via the Locale Home iOS grant
  flow, and live device sessions (TLS-pinned, per-call role:ha tokens).
- Local API for the Locale integration: entity schema/state/commands,
  one SSE event stream for all devices, device version passthrough,
  and a firmware OTA proxy.
- SNTP time service and Root-validated telemetry receiver with local
  store-and-forward.

# 04 — Wireless / BLE

## Prior art to specifically re-test
- UBTECH Alpha 1S (2017, IOActive): missing Bluetooth authenticated link
  key — verify TGPRO's BLE pairing actually requires and validates a link
  key, not just "pairing succeeded."
- 2026 public demos: a compromised robot able to spread over short-range
  wireless to a second, fully offline robot — strongly suggests testing
  whether TGPRO's BLE/wifi-direct stack can be used to push *anything*
  (commands, firmware, files) to a peer without authentication.

## Checklist
- [ ] Identify every wireless radio in play: wifi (AP mode? client mode?),
      BLE, any proprietary RF for controllers/E-stops.
- [ ] `bluetoothctl` / `btmgmt` from Kali to enumerate BLE advertising —
      what's the robot broadcasting, what services/characteristics are
      exposed before pairing?
- [ ] Attempt pairing from an unauthorized device — does it succeed
      silently, prompt for confirmation, or require a PIN/passkey tied to
      something you'd actually need owner access to know?
- [ ] Once paired (or even unpaired, if GATT services are world-readable),
      enumerate exposed BLE services/characteristics — any that map to
      control functions, not just telemetry?
- [ ] Check wifi: open AP? WPA2-PSK with a weak/shared password across all
      units? Any admin interface reachable once on the wifi?
- [ ] If there's a companion mobile app, intercept its traffic (Kali +
      a proxy like mitmproxy/Burp) — is the app-to-robot channel encrypted
      and does it verify the robot's identity (or is it trivially
      spoofable / MITM-able, as found in the Alpha 1S/2 research)?

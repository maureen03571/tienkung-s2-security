# Tienkung / Walker S2 TGPRO — Security Assessment

Internal security research and vulnerability assessment of the UBTECH Walker
Tienkung TGPRO humanoid robot platform, performed by the robot's responsible
security owner.

## Scope

**In scope:**
- All compute hosts under direct administrative control: `nvidia` (x2) and
  `x86` boards, accessed via SSH per the authorization below.
- The voice/LLM control pipeline (Qwen, tkvoice, llama.cpp).
- Wireless/BLE pairing and companion-app communication.
- Local network services exposed by any onboard host.
- Firmware/OTA update mechanisms, where accessible from the above hosts.

**Out of scope (unless explicitly added later):**
- UBTECH cloud infrastructure, Freepass backend, or any UBTECH-operated
  servers not owned/administered by this team.
- Any robot not under this team's direct physical/administrative control.
- Social-engineering or physical attacks against UBTECH staff.

## Authorization

This assessment is performed by the individual(s) responsible for the
security of this specific robot, on hardware they are authorized to test.
No testing is performed against systems outside the stated scope, and no
findings are to be publicly disclosed before responsible disclosure to
UBTECH per `SECURITY.md`.

**Fill in before starting real work:**
- Authorizing party / manager:
- Date authorization granted:
- Robot serial number(s) / asset tag(s) in scope:

## Repo structure

| Folder | Purpose |
|---|---|
| `01-recon/` | Network maps, host roles, open ports, running services per host |
| `02-voice-llm-pipeline/` | tkvoice / Qwen / llama.cpp attack surface — can speech or injected input reach actuator commands without authorization? |
| `03-interhost-trust/` | How do nvidia and x86 boards authenticate to each other? Any implicit trust? |
| `04-wireless-ble/` | Pairing flow, authentication, range, companion-app comms |
| `05-firmware-updates/` | Update/OTA mechanism, signature verification, rollback protection |
| `06-findings/` | One file per confirmed finding — see `06-findings/TEMPLATE.md` |
| `07-remediation/` | Tracked fixes and verification that they actually closed the finding |

## Known prior art (not TGPRO-specific, but same vendor / same vuln classes)

- **IOActive, "Hacking Robots Before Skynet" (2017)** — found on UBTECH Alpha
  1S/Alpha 2: missing Bluetooth authenticated link key, no cryptographic
  signature check on companion-app APK updates, app-to-server traffic without
  encryption (enabling MITM update hijacking), and camera/mic access that
  enabled remote spying. UBTECH never published a technical fix advisory.
- **CVE-2025-23183 / CVE-2025-23182** — Open Redirect and Observable
  Discrepancy in UBTECH's **Freepass** platform (likely account/cloud layer,
  not on-robot — verify whether the TGPRO companion app's login flow touches
  Freepass).
- No CVEs or public disclosures currently exist against Walker S2 / Tienkung
  TGPRO specifically — this repo is intended to be among the first
  structured security references for this platform.
- General class: BLE-to-root and voice/wireless-hijack attacks have been
  publicly demonstrated against other 2026-era humanoid/quadruped robots
  (Unitree BLE root access; a robot-to-robot BLE worm demoed by Chinese
  researchers). These validate voice pipeline and BLE as high-priority
  starting points.

## Status

Project started: <!-- fill in date -->
Current phase: Recon

See `06-findings/` for current open items.

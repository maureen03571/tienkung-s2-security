# FINDING-0003 — Unauthenticated WebSocket Control & Telemetry Channel

## Summary
The Walker Tienkung TGPRO exposes a WebSocket endpoint at
`ws://<host>:7123/status` that accepts the robot's full internal control
protocol (protobuf, `proto.com.xhumanoid`) with **no authentication,
handshake, token, or pairing step**. Any client on the same network
segment that can reach TCP/7123 can open the socket and immediately send
any `Command` in the protocol, including read commands (confirmed) and
write/actuation commands (mapped, not yet executed — see Scope).

## Affected Component
- Web UI backend service listening on `tcp/7123`, path `/status`
- Observed on host `192.168.41.1` (role: TBD — fill in which board this is,
  nvidia/orin vs x86)
- Protocol: Protobuf over WebSocket, subprotocol `"status"`
- Message schema: see `evidence/xhumanoid.proto` (reverse-engineered from
  `app-real.js`, module `0089`)

## Severity
**Critical** (preliminary — confirm CVSS after actuation-path testing)
- Confidentiality: full real-time telemetry leak (motors, IMU, force/torque,
  battery, e-stop status, live RC receiver input) — **confirmed**
- Integrity/Availability: command surface includes `CMD_RUN_ACTION`,
  `CMD_RUN_DEVICE_CONTROL`, `CMD_CONNECT_WIFI`, `CMD_RUN_UPLOAD_FILE`,
  `CMD_RUN_VERSION_UPDATE`, `CMD_SET_CONFIG_DATA`/`CMD_SET_PARAMETER` —
  **mapped, auth-bypass not yet exercised on write path**

## Proof of Concept
Using `evidence/poc_get_data_list.py` against `ws://192.168.41.1:7123/status`:

1. Connected with no credentials, headers, or pairing — connection
   accepted immediately.
2. Sent `CMD_GET_STATUS_DATA` (command=13) — received 7634 bytes of live
   telemetry: per-motor pos/speed/current/temperature for every joint,
   IMU orientation/accel, 6-DOF arm force/torque, battery state,
   `/power/board/key_status` (`is_estop: false`, `is_remote_estop: false`,
   `is_power_on: true`), and live `/sbus_data` RC receiver axes.
3. Sent `CMD_GET_DATA_LIST` (command=11) — a request the web UI never
   sends on its own — received `error_code=0` and a full 48-entry device
   inventory (every motor ID + Chinese joint description, plus
   `orin_slam`, `orin_llm`, `x86`, `battery`, `remote_control`, etc.)

This confirms the channel accepts arbitrary structured commands from the
reverse-engineered protocol, not just the one call the official UI issues
automatically — i.e., this is a general-purpose unauthenticated control
API, not a narrow telemetry display quirk.

## Attack Scenario
1. Attacker on the same LAN/Wi-Fi as the robot (or within BLE/AP range —
   see `04-wireless-ble/`) connects to port 7123.
2. Sends `CMD_GET_STATUS_DATA` in a loop, monitoring
   `is_estop`/`is_remote_estop` to know exactly when the robot is live and
   unprotected.
3. At the opportune moment, sends `CMD_RUN_ACTION` with an `ActionType`
   such as `ACT_MOVE_FORWARD` or any arm/leg action — **not yet executed
   in testing; mapped via schema only** (see Scope/Next Steps).

## Root Cause
No authentication, authorization, or session/token validation on the
WebSocket upgrade or on individual `Request` messages. The protocol
relies entirely on network-level trust (i.e., "if you can reach port
7123, you're trusted").

## Scope / What Has NOT Been Done
- No `CMD_RUN_*` (actuation/write) command has been sent yet. This finding
  is based on: (a) confirmed unauthenticated read access, (b) static
  mapping of the write command set from the client-side protocol
  definition. Executing a write command requires physical safety
  precautions (robot secured/stood down, spotter present, clear
  workspace) before proceeding — see capstone safety notes.
- Firmware/update integrity (`CMD_RUN_VERSION_UPDATE`) not yet tested.
- Whether `7123` is reachable beyond the local subnet (routed, exposed via
  cloud relay, etc.) not yet determined.

## Remediation (preliminary)
- Require authentication on WebSocket upgrade (token, mutual TLS, or at
  minimum a shared secret validated before any `Request` is processed).
- Move to `wss://` (TLS) to prevent passive sniffing of telemetry and
  command traffic on the local network.
- Apply least-privilege command authorization — a client authenticated
  for telemetry read should not automatically be able to send
  `CMD_RUN_ACTION`/`CMD_CONNECT_WIFI`/`CMD_RUN_UPLOAD_FILE`.
- Rate-limit and log connections to 7123; alert on connections from
  unexpected source IPs/MACs.

## References
- `evidence/xhumanoid.proto` — reconstructed schema
- `evidence/poc_get_data_list.py` — PoC script
- Prior art: Unitree BLE root-access disclosure; recent humanoid/quadruped
  voice-hijack research (robot-to-robot BLE worm) — see area notes for links

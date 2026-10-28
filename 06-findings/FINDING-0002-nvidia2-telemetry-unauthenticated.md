# FINDING-0002: Unauthenticated binary telemetry stream on nvidia-2:25786

- **Date found:** 2026-10-28
- **Found by:** maureen03571
- **Host/component affected:** nvidia-2 (192.168.41.3:25786)
- **Status:** Confirmed (access gap); payload schema still unknown
- **Severity:** Medium-High, pending schema and write-capability testing
- **CWE:** CWE-306 (Missing Authentication for Critical Function)

## Summary
Raw TCP connection to port 25786 immediately returns a fixed 46-byte binary
payload, no handshake or credential required.

## Technical detail
python3 -c “
import socket
s = socket.create_connection((‘192.168.41.3’, 25786), timeout=5)
data = s.recv(4096)
print(data.hex())
print(len(data), ‘bytes’)
“
Output (identical on every connection attempt, including nmap's blind
probe):
000018040000000000000400400000000500400000000600004000fe0300000001000004080000000000003f0001
46 bytes, fixed length, sent unprompted on connect.

## Impact
Unknown pending schema decode — likely telemetry (joint state, IMU,
battery) given the host's role as body-control. At minimum this is an
information-disclosure finding (any device on the network can passively
observe robot state). Severity increases significantly if the same port
also accepts writes (would allow unauthenticated command injection into
motor control).

## Evidence
See hex dump above.

## Recommended fix
TBD pending schema confirmation — likely: bind to localhost if only
on-host consumers need it, or add authentication/mutual TLS if it must
be network-reachable.

## Next steps
- [ ] Decode the 46-byte schema (cross-reference dashboard JS source,
      01-recon/app.js once pulled)
- [ ] Test whether the port accepts writes, not just reads (carefully —
      in a controlled test, not against a live/active robot)
- [ ] Confirm whether this is read-only telemetry or a bidirectional
      control channel

## Disclosure status
- [ ] Reported to UBTECH
- [ ] UBTECH acknowledged
- [ ] Fix confirmed
- [ ] Moved to 07-remediation/

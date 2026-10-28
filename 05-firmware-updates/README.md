# 05 — Firmware / OTA Updates

## Prior art to specifically re-test
UBTECH's Alpha 1S/Alpha 2 companion app installed updates with **no
cryptographic signature verification**, over a connection with **no
encryption**, allowing MITM substitution of the update package. This is the
single most directly-transferable historical finding — check whether TGPRO
inherited the same update architecture pattern (even if the specific app is
different).

## Checklist
- [ ] Identify every update path: companion app OTA, local update via
      SSH/USB, any auto-update daemon running on nvidia/x86 hosts.
- [ ] For each path: is the update package signed? Is the signature
      actually verified before flashing/installing, or just present-but-
      unchecked?
- [ ] Is the update fetched over TLS with certificate validation, or could
      it be MITM'd on the local network?
- [ ] Is there rollback protection (can an attacker push an older,
      known-vulnerable firmware version)?
- [ ] Who/what has write access to the update mechanism on-host — does it
      run as root, and could a lower-privileged compromise escalate via
      the updater?
- [ ] If you have physical/SSH access already as owner: can you craft and
      install an unsigned "update" yourself? (Confirms the gap rather than
      assuming it.)

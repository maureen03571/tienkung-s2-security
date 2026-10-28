# 01 — Recon

Goal: build an accurate map of what's actually running before touching
anything else.

## Checklist
- [ ] Document the three hosts: role, IP, OS/kernel version, how you access
      each (SSH key vs password, which user).
- [ ] From Kali, `nmap -sV -O` (or `-A` for more detail) each host's IP —
      save raw output here as `nmap-<host>.txt`.
- [ ] From each host itself (via your existing SSH access), list listening
      services: `ss -tulnp` (or `netstat -tulnp` if ss isn't available).
      This catches services bound to localhost-only that nmap from outside
      won't see — often the most interesting ones (internal APIs trusting
      "it's local, so it's safe").
- [ ] List running processes relevant to robot control:
      `ps aux | grep -iE 'ros|tkvoice|llama|bodyctrl|proc_manager'`
- [ ] Check for default/weak credentials on any exposed service (SSH
      password auth enabled? default user/pass on any web UI?).
- [ ] Note any shared filesystem, shared ROS/DDS domain, or other channel
      the three hosts use to talk to each other — this feeds into
      `03-interhost-trust/`.
- [ ] Note the robot's own wifi/BLE presence — SSID, whether it's an AP
      itself or a client, BLE advertising name — feeds into `04-wireless-ble/`.

## Files to produce here
- `host-inventory.md` — table of the 3 hosts and their roles
- `nmap-<host>.txt` — raw scan output per host
- `listening-services-<host>.txt` — `ss -tulnp` output per host

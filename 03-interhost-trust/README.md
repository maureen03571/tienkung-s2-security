# 03 — Inter-host Trust (nvidia <-> x86)

## Core question
If an attacker compromises the weaker of your hosts, do they automatically
get the stronger one too?

## Checklist
- [ ] How do the hosts communicate — ROS/DDS topics, a custom websocket
      (you've already seen `bodyctrl_msg`-style messaging), shared NFS/SMB
      mount, SSH key trust between boards?
- [ ] Is that channel authenticated, or does "being on the local network /
      local bus" count as trusted?
- [ ] If ROS1: is `ROS_MASTER_URI` reachable from outside the robot's
      internal network? ROS1's computational graph has historically had
      *no* built-in authentication (this was the root cause of several
      critical CVEs on other robot platforms, e.g. MiR mobile robots) —
      worth checking explicitly even if you expect it's "internal only."
- [ ] If ROS2/DDS: is DDS Security (SROS2) actually enabled, or running
      with default/no access control? Unencrypted DDS traffic can leak
      sensor data and allow traffic injection.
- [ ] Do the two nvidia hosts trust each other the same way, or is there an
      asymmetry (one is more privileged)?
- [ ] SSH key reuse: is the same key/credential valid across all three
      hosts? If so, compromising the weakest one's key = compromising all.

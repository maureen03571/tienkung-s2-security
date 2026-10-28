# Disclosure & Handling Policy

This repo may at times contain unpatched vulnerability details affecting a
commercially deployed humanoid robot. Treat it accordingly.

## Repo visibility

- **Keep this repository PRIVATE on GitHub.** Do not make it public, do not
  fork it to a personal public account, do not paste finding details into
  public issues, gists, or chat tools outside your organization.
- If any collaborators are added, limit to people who actually need to see
  findings (your security team / direct manager), not the whole org.

## Vendor disclosure

- If a finding is confirmed to affect current production TGPRO units (not
  just your test unit, if that distinction is knowable), consider whether
  UBTECH needs to be notified — especially for anything remotely exploitable
  (network, BLE, voice pipeline) rather than requiring physical/root access
  you already have as the owner.
- A reasonable disclosure window to adopt, modeled on common industry
  practice (e.g. Alias Robotics' RVD, Google Project Zero): **90 days** from
  vendor notification before any public write-up, extendable if the vendor
  commits to a concrete patch date.
- Findings that only matter because *you* have privileged
  owner/administrator access to your own unit (e.g. "root SSH access lets me
  do X") are not vendor disclosures — they're just things to lock down on
  your own deployment.

## Severity triage (quick reference)

Use something like Alias Robotics' RVSS or just CVSS as a starting point:
- **Critical** — remote, unauthenticated, leads to arbitrary actuator
  control, data exfiltration of sensitive data, or persistent compromise.
- **High** — remote but requires some precondition (same network, paired
  device, physical proximity for BLE).
- **Medium** — local/physical access required, or limited impact (info
  disclosure without direct actuator control).
- **Low** — requires existing privileged access, or impact is minor.

## Findings handling

- Every finding gets a file in `06-findings/` using `TEMPLATE.md`.
- Do not commit exploit code, credentials, raw captured traffic containing
  secrets, or anything that would let someone else reproduce an attack
  against a unit they don't own, without first sanitizing it.
- Once a finding is remediated and verified, move the summary (not the full
  technical detail) into `07-remediation/` with a link back to the original
  finding.

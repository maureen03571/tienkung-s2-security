# FINDING-0001: Ollama API exposed unauthenticated on network interface

- **Date found:** 2026-10-28
- **Found by:** maureen03571
- **Host/component affected:** nvidia-1 (192.168.41.2:11434)
- **Status:** Confirmed
- **Severity:** High (possibly Critical pending write-endpoint testing)
- **CWE:** CWE-306 (Missing Authentication for Critical Function)

## Summary
Ollama LLM server is bound to 0.0.0.0:11434 and reachable from the network
with zero authentication required.

## Technical detail
From Kali (not SSH'd into any robot host):
curl http://192.168.41.2:11434/api/tags
Returned the full model inventory with no credential:
- qwen2.5:1.5b
- qwen3-vl:8b (vision + tools + thinking capable)
- qwen3:8b (tools + thinking capable)

Confirmed via `ss -tulnp` on the host itself that Ollama listens on `*:11434`
(all interfaces), not `127.0.0.1:11434`.

## Impact
Any device on the local network can:
- Query the LLM directly via /api/generate or /api/chat, bypassing
  whatever gating (wake-word, intent filtering) the normal tkvoice
  pipeline applies before input reaches the model.
- Potentially pull a different/malicious model or delete existing models
  via /api/pull and /api/delete (write endpoints not yet confirmed —
  see Next steps).

## Evidence
Raw curl output saved in 01-recon/ (or paste here once collected).

## Recommended fix
Set `OLLAMA_HOST=127.0.0.1` so Ollama only accepts local connections, and
have tkvoice (which presumably runs on the same host) connect via
localhost. If cross-host access from nvidia-2/x86 is genuinely required,
put it behind a reverse proxy with API-key auth, or restrict via firewall
rules to only the specific hosts that need it.

## Next steps
- [ ] Confirm whether /api/pull and /api/delete are reachable without auth
      (test read-only first — e.g. a malformed request — before anything
      destructive)
- [ ] Trace whether tkvoice itself talks to this over localhost or over
      the network (determines whether binding to 127.0.0.1 alone fixes it)

## Disclosure status
- [ ] Reported to UBTECH
- [ ] UBTECH acknowledged
- [ ] Fix confirmed
- [ ] Moved to 07-remediation/

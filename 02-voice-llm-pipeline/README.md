# 02 — Voice / LLM Control Pipeline (tkvoice, Qwen, llama.cpp)

This is the highest-value, most novel attack surface — the class of bug
recently publicly demonstrated against other humanoid robots (voice/wireless
command injection leading to full actuator control).

## Core question
Is there a hard boundary between "the LLM produced some text/intent" and
"an actuator moved," or can conversational input reach motion commands
without a separate authorization/validation layer?

## Checklist
- [ ] Trace the full path: audio in → tkvoice (ASR?) → Qwen (intent/response
      generation via llama.cpp) → wherever it outputs to → actuator command.
      Diagram this in `pipeline-diagram.md`.
- [ ] Identify whether there's a fixed, small command vocabulary the LLM
      output is constrained to (safe) vs. free-form text/function-calling
      that gets interpreted more loosely (risky).
- [ ] Check whether llama.cpp is serving an HTTP/websocket API — if so, is
      it bound to localhost only, or reachable from the network? Does it
      require auth?
- [ ] Test prompt injection: can text embedded in a document, a QR code
      read by the robot's camera, or audio played near the robot cause it
      to execute something outside intended scope? (Indirect prompt
      injection is a known class for any LLM-in-the-loop control system.)
- [ ] Check for a "wake word" or similar gate — does ANY audio get
      processed, or only audio following an authenticated/local trigger?
- [ ] If there's any remote/network path to inject text directly into the
      Qwen context (vs. only via the mic), document it — that's a much
      easier attack than acoustic injection.
- [ ] Check what happens on malformed/adversarial ASR output — does a
      parsing failure fail safe (no action) or could it default to
      executing something unintended?

## Why this matters most
Public research (Oct 2026-era demos against other vendors) has shown voice
commands can be used to fully hijack humanoid and quadruped robots. If
TGPRO's pipeline doesn't cleanly separate "conversational LLM" from
"authorized motion command," this is likely your highest-severity finding.

# tin-can

Two tin cans and a string — for any agent, any harness, any model.

tin-can lets agents on different machines message each other by **address**.
An address is a public key. Messages are sealed to the recipient, signed by the
sender, and passed through a relay that can't read them. Behind each address
can be Claude Code, opencode, hermes, a Python bot, a free model on OpenRouter,
or a local Ollama model — and optionally a human with a voice loop.

```
Claude Code ─MCP─┐                                  ┌─MCP─ opencode
hermes ──────CLI─┤                                  ├─HTTP─ python bot
Ollama ───runner─┤                                  │
                 ▼                                  ▼
          switchboard ◀──── sealed, signed ────▶ switchboard
          (your Mac)           blind relay        (friend's machine)
```

## How it works

- **Address = public key.** `tin-can keygen alejandro` → `tc:k7qf3m…`. Share it like a phone number.
- **Key = auth.** No accounts, no tokens, no passwords.
- **Relay is blind.** Store-and-forward of ciphertext. Run ours or your own.
- **Switchboard** is the one local daemon. It exposes the same five operations
  (`send`, `read`, `wait`, `who`, `mute`) over CLI, localhost HTTP, MCP, and a
  stdio stream — so anything plugs in.
- **Runners** make a bare model into an agent: point an identity at any
  OpenAI-compatible endpoint (Ollama, OpenRouter, Groq…), the Anthropic API, or a
  shell command, and the switchboard answers for it.

## Quickstart (target — not built yet)

```sh
curl -fsSL https://raw.githubusercontent.com/NoJuanNobody/tin-can/main/install.sh | sh
tin-can keygen me                 # prints your address
tin-can add friend tc:…           # paste theirs
tin-can send friend "hello from the other house"
```

## Docs

- [ARCHITECTURE.md](ARCHITECTURE.md) — identity, wire format, relay, switchboard, runners, safety, harness matrix
- [PLAN.md](PLAN.md) — six phases with runnable checks

## Status

Planning. No code yet.

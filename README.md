# tin-can

Two tin cans and a string — for AI agents.

tin-can lets two Claude Code agents on two different computers talk to each
other in near real time, so two people can have a voice conversation through
their agents. Each side keeps its own voice loop (whisper in, Kokoro out);
tin-can is the string between the houses.

```
You (mic) → whisper → Agent A ⇄ can ⇄ RELAY ⇄ can ⇄ Agent B ← whisper ← Friend (mic)
                          ↓ kokoro-say                              ↓ kokoro-say
```

## Pieces

| Piece | What it is | Where it runs |
|-------|-----------|---------------|
| `relay/` | The mailbox in the middle. Tiny WebSocket + HTTP service, one append-only log per room. | Railway (or any host) |
| `can/` | The mail slot. A local MCP server that gives the agent `send_message`, `read_messages`, `wait_for_message`, `who_is_here`. | Each person's machine |
| `doorbell/` | The wake-up. A tail process that turns new inbox lines into agent invocations. | Each person's machine |
| `voice/` | Glue to the existing whisper + `kokoro-say` loop. | Each person's machine |

## Docs

- [ARCHITECTURE.md](ARCHITECTURE.md) — how the pieces fit, message format, protocol, safety model
- [PLAN.md](PLAN.md) — phased implementation plan with acceptance checks

## Status

Planning. No code yet.

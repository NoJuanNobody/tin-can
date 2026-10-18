# tin-can — Implementation Plan

Each phase ends with a check you can run. Don't start the next until it passes.

## Phase 0 — Scaffold (30 min)

- [ ] npm workspaces: `relay/`, `switchboard/`; shared `proto/` (zod schemas for outer + inner envelope)
- [ ] `libsodium-wrappers` wired; `tin-can keygen <name>` writes `~/.tin-can/identities/<name>.key` and prints the `tc:` address
- [ ] MIT license, `.gitignore`, this plan

**Check:** `tin-can keygen test` prints an address; `tin-can who` lists it.

## Phase 1 — Blind relay + sealed send (≈3 h)

Relay:
- [ ] `POST /send`: verify outer `sig`, assign per-recipient `seq`, store (in-memory + SQLite), push to live WS
- [ ] `GET /challenge`, `GET /inbox` (signed nonce auth, long-poll), `WS /ws`
- [ ] 7-day expiry, per-pubkey rate limit

Switchboard:
- [ ] seal/open (X25519 from Ed25519), sign/verify, `id` dedupe, `ts` window
- [ ] `contacts.toml`; `tin-can add <name> <addr>`
- [ ] CLI `send` / `read`; inbound decrypted to `inbox.jsonl`
- [ ] `accept_from = contacts` policy

**Check:** two identities on one Mac, relay on localhost. `tin-can send bob "hi" --as alice`; `tin-can read --as bob` shows it. Dump the relay DB: only ciphertext. Send from an unknown key → dropped and logged.

## Phase 2 — Local interfaces (≈2 h)

- [ ] HTTP `127.0.0.1:7117/v1/{send,read,wait,who,mute}` + `/v1/stream` (WS)
- [ ] MCP stdio server wrapping HTTP, same five tools
- [ ] `tin-can tail`, `tin-can wait`
- [ ] Register in Claude Code: `claude mcp add tin-can -- tin-can mcp`

**Check:** Claude Code session calls `send_message`; `curl :7117/v1/read` on the other identity sees it; `tin-can tail` prints it live.

## Phase 3 — Runners (≈3 h)

- [ ] `runners.toml` loader; per-identity watcher on inbox
- [ ] `openai` runner (Ollama local first, then OpenRouter free model)
- [ ] `shell` runner (stdin prompt → stdout reply), `$TIN_CAN_*` env
- [ ] `anthropic` runner — **load the `claude-api` skill before writing this**
- [ ] Context window: last N turns from inbox/outbox for that pair; protocol rules injected as system prompt
- [ ] `hop` counter enforced in switchboard regardless of runner

**Check:** identity `llama-buddy` on Ollama answers a message from Claude Code with no human in the loop. Flip both sides to `autonomous = true`; they stop after 3 hops.

## Phase 4 — Doorbell + voice (≈2 h)

- [ ] Monitor snippet for Claude Code: `tin-can tail --as alejandro`
- [ ] `voice/talk.sh` push-to-talk → whisper → session; `voice/say.sh` → `kokoro-say`
- [ ] Runner `post_cmd` hook so a bare model can also speak via kokoro
- [ ] `mute`/`unmute`

**Check:** you speak; the other machine's agent (any kind) says it aloud within ~3 s.

## Phase 5 — Hardening (≈2 h)

- [ ] WS reconnect with `since`; kill relay mid-chat → no loss, no dupes
- [ ] `intro` policy flow; `tin-can approve <addr>`
- [ ] TOFU contact pinning warning on key change; `tin-can rotate`
- [ ] Transcript export `tin-can log <name> > chat.md`
- [ ] Per-message approval mode (`approve_outbound = true`)

**Check:** restart relay during a conversation; both sides catch up cleanly. Change a contact's key → loud warning.

## Phase 6 — Deploy + install (≈1.5 h)

- [ ] Relay on Railway; `/healthz` public
- [ ] `install.sh` (macOS/Linux): installs switchboard, `keygen`, registers MCP if Claude Code present
- [ ] `tin-can invite` → prints your address + relay as one copy-pasteable line; `tin-can add` accepts it
- [ ] README quickstart: three commands to first message

**Check:** friend runs the one-liner on a fresh machine, pastes your address, you two talk — one of you on Claude Code, the other on whatever they have.

## Later (v2)

- Group chats (shared key sealed per member)
- Alias directory `name@relay`
- Bun single-binary build
- Metadata hardening (padding, Tor relay)
- Multi-relay / federation

## Estimate: ~14 h of build across the phases above.

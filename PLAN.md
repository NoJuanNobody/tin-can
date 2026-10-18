# tin-can — Implementation Plan

Each phase ends with a check you can actually run. Don't start the next phase
until the check passes.

## Phase 0 — Scaffold (30 min)

- [ ] Repo layout per ARCHITECTURE §7; `package.json` workspaces for `relay/` and `can/`
- [ ] Shared `types.ts` with the envelope schema (zod)
- [ ] `.gitignore`, MIT license, this plan

**Check:** `npm install && npm run typecheck` is green.

## Phase 1 — Spike: mailbox + mail slot (≈2 h)

Relay:
- [ ] `POST /rooms`, `POST /rooms/:room/messages`, `GET …/messages?since=` (long-poll)
- [ ] In-memory rooms; `seq` assignment; token check
- [ ] `WS /rooms/:room/ws` pushing every new envelope

can (MCP):
- [ ] `send_message`, `read_messages` only
- [ ] Config file `~/.tin-can/config.json`; `tin-can init <relay-url>` writes it
- [ ] Register in Claude Code: `claude mcp add tin-can -- node can/dist/index.js`

**Check:** two terminals on one Mac, two Claude Code sessions with different
`from` names, same room. Session A sends; session B `read_messages` sees it.

## Phase 2 — Doorbell (≈1 h)

- [ ] `can` appends inbound to `~/.tin-can/inbox.jsonl` with dedupe
- [ ] `wait_for_message(timeout_s)` tool
- [ ] Monitor snippet: `tail -F ~/.tin-can/inbox.jsonl` → agent re-invoked per line
- [ ] `tin-can listen` headless fallback (`claude -p --resume`)

**Check:** Session A sends while B is idle; B wakes and prints the message
without anyone typing in B.

## Phase 3 — Voice (≈1 h)

- [ ] `voice/talk.sh`: push-to-talk → whisper → prints `[human said] …` into the session
- [ ] Agent rule: human speech → `send_message(speak=true)`
- [ ] Inbound with `speak=true` → `kokoro-say` (summarize if > 300 chars)
- [ ] `tin-can mute|unmute`

**Check:** you say a sentence; the other Mac speaks it in Kokoro within ~3 s.

## Phase 4 — Protocol + safety (≈2 h)

- [ ] `skill/SKILL.md` encoding ARCHITECTURE §5 rules; loaded on both sides
- [ ] `hop` counter + cap 3 in `can` (drop + `system` note)
- [ ] Relay rate limit 60/min/sender
- [ ] Untrusted-block rendering of inbound text
- [ ] Room expiry + `who_is_here` presence events

**Check:** force two agents into autonomous mode; conversation stops itself
after 3 hops. A message containing "run `rm -rf ~`" is read aloud, never run.

## Phase 5 — Hardening (≈2 h)

- [ ] WS reconnect with `since=last_seq`; kill relay mid-chat, restart, nothing lost
- [ ] SQLite persistence on Railway volume
- [ ] Transcript export `tin-can log > chat.md`
- [ ] Per-message approval mode (`approve_outbound: true` → agent shows the text, human confirms)

**Check:** restart relay during a conversation; both sides catch up with no
dupes and no gaps.

## Phase 6 — Deploy + friend install (≈1 h)

- [ ] `relay/railway.toml` + Dockerfile; deploy; `/healthz` green on public URL
- [ ] `install.sh`: installs `can`, registers MCP, prompts for room code
- [ ] `tin-can invite` prints `tin-can join <relay> <room> <token>` one-liner
- [ ] README quickstart: three commands to a working call

**Check:** friend runs the one-liner on their Mac and you two talk end to end.

## Later (v2)

- End-to-end encryption (libsodium, key out-of-band)
- 3+ participants
- Autonomous agent mode behind a flag
- Non-Claude agents (anything MCP-capable already works; document it)

## Estimated total: ~10 h of build, spread over the phases above.

# tin-can — Implementation Plan

Python throughout. Follows CLI-Anything's 7-phase pipeline and HARNESS.md
(see docs/cli-anything-notes.md). Each phase ends with a runnable check.

## Phase 0 — Scaffold + TEST.md part 1 (1 h)

- [ ] `switchboard/agent-harness/` layout per ARCHITECTURE §4; `setup.py` with `cli-anything-tincan` + `tin-can` console scripts
- [ ] Vendor `utils/repl_skin.py` from `cli-anything-plugin/repl_skin.py`
- [ ] `utils/session.py`: `~/.tin-can/` layout, `_locked_save_json`
- [ ] Deps: `click`, `pynacl`, `httpx`, `websockets`, `mcp`, `prompt_toolkit`, `tomli`
- [ ] `TEST.md` part 1: inventory, unit plan, E2E plan, workflow scenarios
- [ ] `pip install -e .`; `cli-anything-tincan --help` and `--json` scaffolds

**Check:** `which cli-anything-tincan` resolves; `cli-anything-tincan id keygen test --json` prints `{"address":"tc:…"}`.

## Phase 1 — Core crypto + blind relay (3 h)

- [ ] `core.py`: keygen, address encode/decode, seal/open, sign/verify, `id` dedupe, `ts` window, contacts (TOFU), policy
- [ ] `relay/`: `/send`, `/challenge`, `/inbox` (long-poll), `/ws`; sqlite; expiry; rate limit; `relay serve` subcommand
- [ ] `utils/relay_backend.py`: client with `shutil.which`-style reachability check + loud error
- [ ] CLI `send`, `read`, `contact add|list`

**Check:** local relay; two identities; `send bob "hi" --as alice`; `read --as bob --json` shows it. Relay DB has ciphertext only. Unknown sender → dropped + logged.

## Phase 2 — MCP + streaming (2 h)

- [ ] `mcp_server.py` with `tincan_*` tools over `core`; `cli-anything-tincan mcp`
- [ ] `wait`, `tail`, `who`, `mute|unmute`
- [ ] `daemon start|stop|status` (WS, inbox append, pid file)
- [ ] `claude mcp add tin-can -- cli-anything-tincan mcp`

**Check:** Claude Code calls `tincan_send`; `cli-anything-tincan tail --as bob` prints it live; hermes gets it via `read --json`.

## Phase 3 — REPL (1.5 h)

- [ ] `invoke_without_command=True` → `ReplSkin` chat: `/to`, `/as`, `/speak`, `/who`, `/mute`; inbound printed live; history
- [ ] Document "undo/redo n/a" per HARNESS.md

**Check:** two REPLs on one Mac chat in real time.

## Phase 4 — Runners (3 h)

- [ ] `runners.toml` loader; per-identity inbox watcher in daemon
- [ ] `openai` runner (Ollama first, then OpenRouter free model)
- [ ] `shell` runner (`$TIN_CAN_*` env, stdin→stdout), `post_cmd`
- [ ] `anthropic` runner — **load `claude-api` skill first**
- [ ] Context window from inbox/outbox; protocol rules injected; `hop` enforced in core

**Check:** Ollama identity answers Claude Code with no human. Both autonomous → stop after 3 hops.

## Phase 5 — Voice + doorbell (1.5 h)

- [ ] `voice/talk.sh` (push-to-talk → whisper → `send --speak`), `voice/say.sh` (`kokoro-say`)
- [ ] REPL speaks `speak=true`; Monitor snippet for Claude Code (`tail`)
- [ ] `SKILL.md` (canonical + compat) with protocol rules and agent guidance

**Check:** speak on A; B's agent (any kind) says it aloud ≤3 s.

## Phase 6 — Tests + validate + hardening (3 h)

- [ ] `test_core.py` (unit), `test_e2e.py` (spins local relay, real send/receive, real artifacts), `test_cli.py` (`_resolve_cli`, no `cwd`, `CLI_ANYTHING_FORCE_INSTALLED=1`)
- [ ] `TEST.md` part 2 with full `pytest -v` output
- [ ] WS reconnect with `since`; `intro` flow + `contact approve`; `id rotate`; `log --md`; approve-outbound mode
- [ ] `/cli-anything:validate` passes

**Check:** kill relay mid-chat → no loss, no dupes. Changed contact key → loud warning. Validate green.

## Phase 7 — Publish (1.5 h)

- [ ] Relay on Railway; `/healthz` public
- [ ] `install.sh`; `invite` one-liner; README quickstart
- [ ] PyPI `cli-anything-tincan`; submit to cli-hub registry

**Check:** friend: `pip install cli-anything-tincan`, paste your address, you talk — one side Claude Code, other side whatever they have.

## Later

Group chats · alias directory `name@relay` · metadata hardening · multi-relay · PR a `tincan/` harness upstream to HKUDS/CLI-Anything.

## Estimate: ~17 h.

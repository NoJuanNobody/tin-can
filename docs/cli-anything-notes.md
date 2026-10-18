# CLI-Anything — research notes (2026-10-18)

Source: https://github.com/HKUDS/CLI-Anything (and `cli-anything-plugin/HARNESS.md`,
`cli-anything-plugin/guides/mcp-backend.md`, `cli-hub-meta-skill/SKILL.md`,
`hermes-skill/SKILL.md`).

## What it is

A Claude Code plugin (`/plugin marketplace add HKUDS/CLI-Anything`,
`/plugin install cli-anything`) that runs a 7-phase pipeline — Analyze, Design,
Implement, Plan Tests, Write Tests, Document (+ SKILL.md), Publish — to generate
a Python/Click CLI "agent harness" for any software. 40+ harnesses shipped
(GIMP, Blender, OBS, Ollama, Obsidian, …). Motto: "Making ALL software
agent-native." CLI is treated as the universal interface for both humans and
agents.

Companion: **cli-hub** (`pip install cli-anything-hub`; `cli-hub search`,
`cli-hub install <name>`) — registry so agents on any harness (Claude Code,
Codex, hermes, OpenClaw, Nanobot, opencode) can discover and install CLIs.
Per-harness skill packs exist: `hermes-skill/`, `codex-skill/`,
`opencode-commands/`, `.cursor-plugin/`, `.pi-extension/`.

## HARNESS.md standards (what we must follow)

**Naming**
- PyPI package `cli-anything-<software>`; module `cli_anything.<software>`
- `cli_anything/` has **no** `__init__.py` (PEP 420 namespace); `<software>/` does
- Console script `cli-anything-<software>`; also `python -m cli_anything.<software>`
- Main Click file `<software>_cli.py`; backend wrapper `utils/<software>_backend.py`

**Layout**
```
<software>/agent-harness/
  setup.py
  cli_anything/<software>/
    __init__.py  __main__.py  <software>_cli.py  core.py
    utils/repl_skin.py  utils/<software>_backend.py
    skills/SKILL.md        (compat copy)
    README.md
    tests/ test_core.py test_e2e.py test_cli.py TEST.md
  skills/cli-anything-<software>/SKILL.md   (canonical)
```

**Behaviour**
- Every command supports `--json`; human-readable by default
- `--help` everywhere; `--project <path>` / `--overwrite` where applicable
- Bare invocation → REPL (`invoke_without_command=True`), using the shared
  `ReplSkin` (`print_banner, create_prompt_session, get_input, help, success,
  error, warning, info, status, table, progress, print_goodbye`)
- Session state = JSON file, saved with exclusive lock (`_locked_save_json`:
  open `r+`, lock, truncate inside lock); undo/redo via command pattern where
  the software supports it (50-level)
- Exit 0 success / non-zero failure; "fail loudly" with install instructions
  when a dependency is missing (`shutil.which` + `RuntimeError`)
- Real backend is a **hard dependency**: no fake/degraded mode, E2E tests must
  hit the real thing and produce real artifacts

**SKILL.md** frontmatter `name: "cli-anything-<software>"`, `description:`;
body = prerequisites, syntax, command groups, examples, agent guidance
(`--json`, error handling); self-contained.

**Tests**: `TEST.md` part 1 (plan) before code, part 2 (full `pytest -v`
output + stats) after. Subprocess tests use `_resolve_cli("cli-anything-<software>")`,
never hardcode paths, never set `cwd`; `CLI_ANYTHING_FORCE_INSTALLED=1` in CI.

## MCP guidance (`guides/mcp-backend.md`)

Covers a CLI *consuming* an MCP server: `utils/<service>_backend.py` holds an
async `_call_tool()` that spawns the server over stdio and uses `ClientSession`;
Click commands are thin sync wrappers (`asyncio.run`). Server is spawned per
command (stateless) with an optional `--daemon` mode that keeps one process
alive for latency. Add `mcp>=0.1.0` to `install_requires`. Reference impl:
`browser/agent-harness`.

Nothing in the repo covers a harness *exposing* MCP. We add that ourselves:
`mcp_server.py` next to the Click file, both calling `core.py`, started via
`cli-anything-tincan mcp`.

## What this means for tin-can

- Switchboard becomes **`cli-anything-tincan`** (module `cli_anything.tincan`),
  Python, following HARNESS.md to the letter so it validates with
  `/cli-anything:validate` and can be listed on cli-hub.
- Two interfaces only: **CLI** (incl. `tail`/`wait` for streaming) and **MCP**.
  HTTP localhost API dropped; bots shell out to `--json`.
- Relay also Python (one toolchain).
- REPL = the human's "phone": bare `cli-anything-tincan` opens a chat REPL.
- Harness-agnostic distribution rides CLI-Anything's existing per-harness
  skill packs + cli-hub, instead of us building adapters.

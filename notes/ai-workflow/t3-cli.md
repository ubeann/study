---
title: The t3 CLI
kind: concept
area: ai-workflow
date: 2026-10-21
status: verified
diagram: diagrams/t3-cli.html
sources:
  - https://github.com/pingdotgg/t3code
tags: [t3-code, cli, agent-harness, claude-code, codex]
---

## Summary

`t3` is the command that installs, runs and maintains the T3 Code server, the "agent harness control surface" that drives Codex, Claude Code, Cursor, Grok Build, OpenCode and Antigravity from a web, desktop or mobile client. It does not send prompts to agents; the server does, and outside agents reach the server's features over MCP. Local binary checked for this note: `~/.t3/bin/t3 --version` prints `t3 v0.1.1-nightly.20261020.3438`.

## The problem it solves

Each agent CLI (`codex`, `claude`, `agent`, `grok`, `opencode`) has its own login and its own terminal session. Running several of them across projects, keeping them alive when the laptop lid closes, and reaching them from a phone means hand-rolling a server, auth, pairing and updates. `t3` packages that: one command starts the server, one installs it as a background service, one updates it.

## How it works

The README path is `curl -fsSL https://t3.codes/install.sh | sh`, then `t3` to start the server and open the local web app (`npx t3@latest` tries it without installing). `t3 --help` on the local binary lists these subcommands:

| Command | What it does (from `--help`) |
|---|---|
| `t3 start [cwd]` / `t3 serve [cwd]` | Run the server; `serve` skips the browser and prints headless pairing details. Flags: `--port`, `--host`, `--mode web\|desktop`, `--base-dir`, `--tailscale-serve` |
| `t3 app [path]` | Open a project in the running desktop app |
| `t3 pair` | Mint a pairing token for a running server and print it as a QR code; `--scope`, `--ttl` (default 5 minutes), `--tailscale` |
| `t3 auth pairing\|session` | Manage pairing tokens and bearer sessions for headless deployments |
| `t3 project add\|remove\|rename` | Manage projects |
| `t3 service install\|restart\|uninstall\|status` | Run T3 Code as a background service for this user |
| `t3 update [version]` | Download a newer t3 and switch to it; `--channel stable\|nightly\|preview` |
| `t3 uninstall` | Remove service, launcher and downloaded versions; projects and threads are kept |
| `t3 theme set\|clear\|show` | Environment-wide theme defaults |
| `t3 trace summary` | Counts, rates and latency from the local trace file |
| `t3 doctor` | Inspect and repair the data directory; `--fix`, `--dry-run` |
| `t3 triage` | Investigate a T3 problem with `--agent claude\|codex` and help file an issue |
| `t3 connect login\|link\|publish\|status\|unlink\|logout` | Set up T3 Connect for phone access |
| `t3 browser setup` | Prepare a Linux host for T3's headless browser |

Global flags: `--wizard`, `--completions <shell>`, `--log-level`.

Order of a typical setup: install, `t3 service install`, `t3 app .` to add the repo, `t3 pair` or `t3 connect link` for another device, `t3 update` when a release lands, `t3 doctor` when something breaks.

## When to use it, when to skip it

| Use it when | Skip it when |
|---|---|
| You run more than one agent or more than one account per provider (README lists multiple Codex and Claude accounts) | One agent, one short task: its own CLI is enough |
| You want agents reachable from a phone or another machine | You never leave the terminal |
| You want the server alive without the desktop app open (`t3 service`) | You are fine starting `t3` by hand each time |
| You want other agents (Claude Code, Codex, ChatGPT) to use T3 features over MCP | You are not using MCP at all |

## Diagram

[diagrams/t3-cli.html](../../diagrams/t3-cli.html): what `t3` manages versus what the agents do, the lifecycle commands in order, and the sources of truth for each. It does not show the MCP tool list or per-provider settings.

## Sources

- https://github.com/pingdotgg/t3code: backs the description, supported providers and login commands, install commands, `t3 service install`, `t3 update`, MCP, multiple accounts, remote access, and the "very very early" warning.
- Local binary `~/.t3/bin/t3` (`t3 v0.1.1-nightly.20261020.3438`), `t3 --help` and each subcommand's `--help`, run 2026-10-21: backs the command table and flags.

---
title: CLIProxyAPI vs T3 Code
kind: comparison
area: ai-workflow
date: 2026-10-21
status: draft
diagram: diagrams/cliproxyapi-vs-t3-code.html
sources:
  - https://github.com/router-for-me/CLIProxyAPI
  - https://raw.githubusercontent.com/router-for-me/CLIProxyAPI/main/cmd/server/main.go
  - https://raw.githubusercontent.com/router-for-me/CLIProxyAPI/main/config.example.yaml
  - https://help.router-for.me/
  - https://github.com/pingdotgg/t3code
  - https://code.claude.com/docs/en/legal-and-compliance
  - https://rogs.me/2026/02/use-your-claude-max-subscription-as-an-api-with-cliproxyapi/
unverified:
  - "Timeline of Anthropic's enforcement against subscription tokens in third-party tools (reported as 2026-01-09 rejection and rollback, 2026-02-19 terms update, 2026-04-04 enforcement). Only the April date appears on a fetched page, and that page is a third-party guide, not an Anthropic page; Anthropic's legal page states the policy without a date."
  - "Exact endpoint paths (/v1/chat/completions, /v1/responses, /v1/messages). The project docs name the protocol families but no fetched page lists the paths."
tags: [cliproxyapi, t3-code, proxy, subscriptions, claude-code]
---

## Verdict

They sit on different layers. CLIProxyAPI is a Go proxy that turns CLI OAuth logins into an OpenAI/Gemini/Claude/Codex-compatible HTTP API with round-robin across accounts. T3 Code is a control surface that drives the official agent CLIs, each logged in through its own vendor flow. If you already work in T3 Code, the proxy only adds something when you need a plain API endpoint for scripts or an IDE plugin; the deciding factor is that Anthropic's own policy forbids routing requests through Free, Pro or Max credentials outside its native apps, so the Claude route is the one not to take.

## Side by side

| | CLIProxyAPI | T3 Code |
|---|---|---|
| Model | A local server (config default `port: 8317`, credentials in `auth-dir: "~/.cli-proxy-api"`) that translates request formats and attaches a stored OAuth token | A server plus web, desktop and mobile clients that start and supervise agent CLIs; outside agents connect over MCP |
| Workflow | Log in once per account (`--claude-login`, `--codex-login`, `--antigravity-login`, `--kimi-login`, `--xai-login`, `--devin-login`, `--meta-login`), point any SDK or tool at the proxy | Install an agent CLI, run its own login (`codex login`, `claude auth login`, `agent login`, `grok login`, `opencode auth login`), add a project, work in threads |
| Tooling and ecosystem | Providers: Claude, OpenAI/Codex, Gemini (API, AI Studio, Vertex, Gemini CLI, Antigravity), xAI Grok, Kimi, Meta Muse, Devin, OpenAI-compatible upstreams; Go SDK; MIT | Providers: Codex, Claude Code, Cursor, Grok Build, OpenCode, Antigravity; multiple Codex and Claude accounts; background service; "very very early. Expect bugs." |
| Cost or limits | Bills the underlying subscription or key; `strategy: round-robin` (also weighted-round-robin, fill-first) across credentials | Bills each CLI's own login; no API endpoint exposed |

## Where CLIProxyAPI wins

- Any client that only speaks an API protocol (scripts, n8n, IDE plugins) can use CLI-based accounts.
- Several accounts per provider with a routing strategy chosen in config.
- Providers T3 Code does not drive: Kimi, Meta Muse, Devin, Vertex, OpenRouter-style upstreams.

## Where T3 Code wins

- It runs agents, not just text: file edits, shell, threads, projects, remote access from a phone, all through the official CLIs.
- Terms exposure stays low. Anthropic: OAuth "is intended exclusively for purchasers of Claude Free, Pro, Max, Team, and Enterprise subscription plans and is designed to support ordinary use of Claude Code and other native Anthropic applications"; developers may not "route requests through Free, Pro, or Max plan credentials on behalf of their users" and Anthropic "may do so without prior notice." The main CLIProxyAPI setup guide for Claude Max marked itself deprecated on 2026-04-04 for exactly this reason.

## When to pick which

- Need an HTTP endpoint for automation, or a model T3 Code cannot reach: CLIProxyAPI, bound to `127.0.0.1` with client `api-keys` set, and only for providers whose terms allow it.
- Day-to-day agent work across providers and accounts: T3 Code alone.
- Both at once is possible (an agent inside T3 Code pointed at the proxy as a custom provider) but inherits the proxy's terms exposure.

## Diagram

[diagrams/cliproxyapi-vs-t3-code.html](../../diagrams/cliproxyapi-vs-t3-code.html): the two layers in one stack, the proxy's request path, and the pick-one table. It does not show the enforcement timeline, which is unverified.

## Sources

- https://github.com/router-for-me/CLIProxyAPI: backs the description, provider list, round-robin claim, Go SDK, MIT license.
- https://raw.githubusercontent.com/router-for-me/CLIProxyAPI/main/cmd/server/main.go: backs the login flag names.
- https://raw.githubusercontent.com/router-for-me/CLIProxyAPI/main/config.example.yaml: backs `port: 8317`, `auth-dir`, `api-keys`, routing strategies.
- https://help.router-for.me/: backs the supported protocol families (Chat Completions / Response / Gemini / Claude).
- https://github.com/pingdotgg/t3code: backs T3 Code's providers, login commands, MCP, multiple accounts, remote access, early-stage warning.
- https://code.claude.com/docs/en/legal-and-compliance: backs the OAuth and credential-routing policy quotes.
- https://rogs.me/2026/02/use-your-claude-max-subscription-as-an-api-with-cliproxyapi/: backs the 2026-04-04 deprecation of the Claude Max setup guide and `--claude-login` usage.

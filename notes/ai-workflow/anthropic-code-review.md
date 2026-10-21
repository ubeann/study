---
title: How the Claude Code team does code review
kind: article
area: ai-workflow
date: 2026-10-21
status: verified
diagram: diagrams/anthropic-code-review.html
sources:
  - https://claude.dev/blog/solving-code-review-at-anthropic/
unverified:
tags: [code-review, claude-code, ai-agents, merge-queue]
---

Article by Addy Osmani, Morgan Westlee Lunt and Qing Wang, published 2026-10-15. Compared here with a normal "one human reads the diff, then approve" PR review.

## What the article claims

- Claude writes about 90% of Claude Code's own code, so a person reading every line cannot keep up. The team's stated goal is "the minimum amount of human review time that doesn't raise reverts or incidents."
- Review is reframed from "did a human read the code?" to "what could go wrong, and which check, automated or human, should catch it?" It runs continuously, "more like spellcheck while you write."
- Human time is the scarce resource. They track human approvals per PR (median and p95), human comments, and time a PR waits on a person. Revert and incident rates are the guardrail: "if they rise, we haven't saved human time, we've only moved it."
- A change passes layers, not one gate:
  1. Before the PR: Claude runs tests, builds or screenshots and shows the output; a fresh-context subagent reviews the diff. Recurring findings move into `CLAUDE.md` (stack, conventions, goals, non-goals; short) or `REVIEW.md` (written by code owners: what failure looks like in that repo, nit caps, paths to skip, file-and-line evidence).
  2. AI review on every PR: several agents hunt bugs in parallel against the whole repo and the PR's stated intent; a separate group votes on each finding; style, diagnostic and cleanup findings are dropped; findings are tagged Important, Nit or Pre-existing; every finding is fixed or resolved with a reason. Admins set a repo to Quick, Standard or Deep.
  3. Security review that can block, with findings cleared automatically by a fixing commit.
  4. Required CI: lint, types, cross-platform tests, build. The PR template asks how it was tested and how to undo it.
  5. Human approval scaled by risk. Low (docs, copy, internal-only, additive behind a flag): none once checks are green. Medium (ordinary product changes; caching, auto-updater, system prompts): one person. High: two people. The change sets the tier, not the author; any CI workflow file always needs a person; an agent's approval never counts as a human's; the person prompting an agent counts as a required reviewer; approvals cover one commit.
  6. Merge queue re-runs checks on main with the change applied; new behaviour ships behind flags to employees first; public releases go to `latest` before `stable`.
  7. Fast rollback: flags turn off without a deploy, a weekly on-call rotation watches releases. They admit they lack "circuit breakers that tighten approval when those numbers move."
- Agents reviewing agent-written code can loop: one CLI PR went through more than fifty review-and-fix rounds overnight, and about 59% of what the reviewer kept flagging was code present since its first pass. Fixes: a three-round cap, diff-only re-review after the first passes, flagging the class of bug, no repeated findings, nightly sweeps over existing code.
- Incidents become evals: PRs that caused incidents or reverts, confirmed by experts, form a set the reviewer runs against daily; every change to prompts, tools or model is scored against it. Priority is "catching real bugs first and cutting review rounds second."
- Humans still decide whether to build something, review risky code (billing, auto-updater, CI), review architecture, read evidence, and own outcomes: "you land it, you own it."

The draft explainer from the chat labelled four parallel agents (logic, system fit, root cause, intent). The article does not name agent focus areas; those labels were the chat's own examples and are not repeated here.

## Key numbers

| Number | Quote from the article |
|---|---|
| ~90% | Claude writes about 90 percent of Claude Code's own code |
| 2,500 to 30,000+ per day | changes landing in the monorepo, May to September |
| 2x merged, -40% reverts | June to September, merged PRs "more than doubled while the share that had to be reverted fell by more than 40 percent" |
| 16% to 54% | internal PRs getting substantive review comments once the code review preview launched |
| 2 to 3x | Boris Cherny's estimate of the gain from letting Claude verify its own work |
| 1,000 lines | most PRs over this size get findings; most small ones come back clean |
| 65% to 78% | PRs passing after one review-and-fix round, before and after convergence work |
| 8.9% to 1.5% | PRs needing five or more rounds |
| 2.08 to 1.32 | mean reviews to pass |
| 50+ rounds, 59% | one PR's overnight loop; share of repeated findings that were pre-existing code |

## What it leaves out

- How many verifier votes a finding needs before it is posted.
- What each parallel agent focuses on.
- The classifier that sets risk tiers: what it reads and how it is tuned.
- Any automatic brake tied to revert or incident rates (the article says this is still missing).

## Status

In production inside Anthropic's own repos: "We didn't remove review. But we did move most of the reading to machines." The article notes that Code Review's public announcement says approval "is still a human call," while the team's internal repos let owners merge low-risk changes on automated checks alone.

## My takeaways

- The useful shift versus a normal review is the question, not the tooling: name the failure, then pick the cheapest check that catches it.
- The two problems a human-only process never has are review loops between agents and reviewers that never learn. Round caps and incident-derived evals are the answers.
- Cheapest first steps from their checklist: `/code-review` and `/security-review` locally, a short `REVIEW.md`, then an AI reviewer on one repo.

## Diagram

[diagrams/anthropic-code-review.html](../../diagrams/anthropic-code-review.html): one-gate review next to the seven-layer version, the path of one change, and the risk tiers. It does not show the eval loop or the convergence numbers; those are in the tables above.

## Sources

- https://claude.dev/blog/solving-code-review-at-anthropic/: backs every claim and number above.

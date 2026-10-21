---
title: AI workflow tools landscape
kind: concept
area: ai-workflow
date: 2026-10-21
status: draft
diagram: diagrams/ai-workflow-tools-landscape.html
sources:
  - https://cloud.google.com/blog/products/ai-machine-learning/announcing-the-2025-dora-report
  - https://survey.stackoverflow.co/2025/ai
  - https://github.blog/news-insights/octoverse/octoverse-a-new-developer-joins-github-every-second-as-ai-leads-typescript-to-1/
  - https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/
  - https://metr.org/blog/2026-02-24-uplift-update/
  - https://metr.org/blog/2026-05-19-frontier-risk-report/
  - https://arxiv.org/abs/2602.11988
  - https://arxiv.org/html/2602.11988
  - https://agents.md
  - https://agentskills.io
  - https://modelcontextprotocol.io
  - https://www.anthropic.com/news/donating-the-model-context-protocol-and-establishing-of-the-agentic-ai-foundation
  - https://github.com/github/spec-kit
  - https://git-scm.com/docs/git-worktree
  - https://github.com/jj-vcs/jj
  - https://github.blog/changelog/2026-10-06-stacked-pull-requests-generally-available/
  - https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/configuring-pull-request-merges/managing-a-merge-queue
unverified:
  - "Faros AI telemetry: review time +91% and PR size +154% on high-AI-adoption teams (vendor blog, not fetched)"
  - "Cursor acquired Graphite, a stacked-PR tool, in Dec 2025 (news coverage only, not fetched)"
  - "AI review catch rates: 82% on Greptile's own test vs 45% when Augment reran it (vendor benchmarks, not fetched)"
  - "Agent Skills is supported by about 40 clients (agentskills.io only says 'a large number')"
tags: [ai-workflow, merge-queue, stacked-prs, agents-md, agent-skills, mcp, spec-driven, jujutsu, worktrees]
---

## Summary

Eight tools and conventions that sit at fixed points in one AI coding loop: context (AGENTS.md, Agent Skills, MCP), plan (spec-driven development), build in parallel (worktrees, Jujutsu), package (stacked PRs), verify (AI code review), land (merge queue). AI made writing code cheap, so review and landing became the slow part; most of these exist to unblock that. The merge queue is the end of the loop, so this note is the rest of it.

## The problem it solves

The usual way: one developer, one branch, one PR, one human reviewer, in series. That worked when writing code was the bottleneck. The surveys say it no longer is:

- 90% of DORA 2025 respondents (nearly 5,000) "report using AI at work"; the report's line is "AI doesn't fix a team; it amplifies what's already there."
- Stack Overflow 2025: 84% "are using or planning to use AI tools", yet "more developers actively distrust the accuracy of AI tools (46%) than trust it (33%)", and 66% name "AI solutions that are almost right, but not quite" as a frustration.
- GitHub Octoverse 2025: "43.2 million pull requests on average each month (+23% YoY)", 518.7M merged in the year (+29%).

So more code arrives, people trust it less, and every line still has to pass a human. The METR trials show why the date on a claim matters: in July 2025 experienced open-source developers took "19% longer" with AI (16 developers, 246 issues) while believing they were 20% faster; by Feb 2026 METR said a speedup is now "likely" but its sample is "systematically missing developers who have the most optimistic expectations"; its May 2026 report puts late-2025 agents at "small (~4-20%) productivity benefits" and calls that an underestimate.

## How it works

Read as one loop; each stage feeds the next.

| Stage | Concept | What it is, from the primary page |
|---|---|---|
| 01 Context | AGENTS.md | "A simple, open format for guiding coding agents", a "README for agents", "used by over 60k open-source projects", now "stewarded by the Agentic AI Foundation under the Linux Foundation". The ETH Zurich / LogicStar paper (arXiv 2602.11988) found context files do "not generally improve task success rates" while "increasing inference cost by over 20% on average"; developer-written files beat LLM-generated ones "by a significant margin of 7% on average". Keep it short and hand-written. |
| 01 Context | Agent Skills | "a folder containing a SKILL.md file" plus optional scripts and references, loaded by progressive disclosure: at startup only name and description, the full file "when a task matches a skill's description". This is the fix for AGENTS.md costing tokens on every task. |
| 01 Context | MCP | "an open-source standard for connecting AI applications to external systems", "like a USB-C port for AI applications". Donated to the Linux Foundation's Agentic AI Foundation on Dec 9, 2025 with "more than 10,000 active public MCP servers"; adopted by ChatGPT, Cursor, Gemini, Copilot, VS Code. |
| 02 Plan | Spec-driven development | GitHub Spec Kit: "Define what to build and why, then turn the specification into a plan, tasks, and working code" via `/speckit-specify`, `-plan`, `-tasks`, `-implement`. 142k stars, MIT, on 2026-10-21. |
| 03 Build | Git worktrees | "A git repository can support multiple working trees, allowing you to check out more than one branch at a time." One folder per agent, so parallel agents never edit the same files. `git worktree add -b <branch> <path>`. |
| 03 Build | Jujutsu (jj) | "A Git-compatible VCS that is both simple and powerful": the working copy is "a real commit", "there is no explicit index or staging area", and jj "records every operation" so mistakes can be undone. 32k stars, Apache-2.0. Same ideas as GitButler, as a CLI. |
| 04 Package | Stacked PRs | "Break large changes into smaller, focused pull requests that you can review independently and merge together." GitHub-native stacks went GA on 2026-10-06; "a stack now enters and lands through the merge queue as a single merge group", and `gh stack` "now supports Git worktrees". |
| 05 Verify | AI code review | A bot takes the first pass before a human does. The only numbers around are vendor benchmarks (see `unverified`), so judge a tool on your own past bugs. |
| 06 Land | Merge queue | "helps increase velocity by automating pull request merges into a busy branch and ensuring the branch is never broken by incompatible changes." Covered in its own note. |

## Learning path

Each step fixes the bottleneck the previous one creates.

1. **Worktrees + a short AGENTS.md.** Run two or three agents in parallel; keep the context file to facts the agent cannot infer from the code (the arXiv result).
2. **Stacked PRs + the merge queue.** Give each agent a small stack instead of one large PR and let the queue land it as one group.
3. **Skills + MCP.** Move repeated prompts into skills; connect one real data source.
4. **Spec-driven on one real feature,** one task per stacked PR. Skip it for spikes.
5. **AI review bake-off** on ten past bugs. Optional: try jj in a scratch repo.

## When to use it, when to skip it

| Use it when | Skip it when |
|---|---|
| Several agents or people work on one repo at once (worktrees, stacks, queue) | A solo repo with one change in flight; plain branches are enough |
| The same instructions get repeated across sessions (AGENTS.md, skills) | The instruction can be inferred from the code; the paper says it then only costs tokens |
| An agent needs live data or tools (MCP) | A file in the repo answers the question |
| A feature is big enough to need a plan (spec-driven) | A quick experiment or spike |

## Diagram

[diagrams/ai-workflow-tools-landscape.html](../../diagrams/ai-workflow-tools-landscape.html): the six-stage loop with each concept placed on it, the usual serial workflow against the parallel one, and the learning order. It does not show the survey numbers; those are above with their sources.

## Sources

- https://cloud.google.com/blog/products/ai-machine-learning/announcing-the-2025-dora-report: backs 90% adoption, ~5,000 respondents, "amplifies" quote.
- https://survey.stackoverflow.co/2025/ai: backs 84%, 46% distrust vs 33% trust, 66% "almost right".
- https://github.blog/news-insights/octoverse/octoverse-a-new-developer-joins-github-every-second-as-ai-leads-typescript-to-1/: backs 43.2M PRs/month (+23%) and 518.7M/year (+29%).
- https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/: backs 19% slower, 16 developers, 246 issues, perceived 20% speedup.
- https://metr.org/blog/2026-02-24-uplift-update/: backs "likely" speedup and the selection-bias caveat.
- https://metr.org/blog/2026-05-19-frontier-risk-report/: backs ~4-20% benefit, "underestimate".
- https://arxiv.org/abs/2602.11988 and https://arxiv.org/html/2602.11988: back the AGENTS.md findings, +20% cost, 7% gap, authors' affiliations.
- https://agents.md: backs the format description, 60k projects, AAIF stewardship.
- https://agentskills.io: backs the SKILL.md structure and progressive disclosure.
- https://modelcontextprotocol.io: backs the MCP definition and USB-C line.
- https://www.anthropic.com/news/donating-the-model-context-protocol-and-establishing-of-the-agentic-ai-foundation: backs the Dec 9, 2025 donation, 10,000+ servers, adopter list.
- https://github.com/github/spec-kit: backs the SDD description, commands, 142k stars, MIT.
- https://git-scm.com/docs/git-worktree: backs the worktree definition and `add` syntax.
- https://github.com/jj-vcs/jj: backs the jj description, 32k stars, Apache-2.0.
- https://github.blog/changelog/2026-10-06-stacked-pull-requests-generally-available/: backs stacked-PR GA, merge-group behaviour, `gh stack` worktrees.
- https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/configuring-pull-request-merges/managing-a-merge-queue: backs the merge-queue definition.

---
title: Merge queue
kind: concept
area: vcs
date: 2026-10-21
status: draft
diagram: diagrams/merge-queue.html
sources:
  - https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/configuring-pull-request-merges/managing-a-merge-queue
  - https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches
  - https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows
  - https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/collaborating-on-repositories-with-code-quality-features/troubleshooting-required-status-checks
  - https://docs.mergify.com/merge-queue/batches/
  - https://docs.mergify.com/merge-queue/queue-modes
  - https://docs.mergify.com/merge-queue/scopes
  - https://docs.mergify.com/merge-queue/speculative-checks/
unverified:
  - "GitHub's merge queue is free; which plans and repository types (public, organization-owned) can use it. No fetched GitHub page carried a 'Who can use this feature' statement."
  - "Graphite's merge queue handles stacked PRs natively; Trunk offers parallel queues. Neither vendor's docs were fetched."
  - "The CI-minute figures in the diagram (176, 132, 50, 39 min) are illustrative arithmetic on assumed job durations, not measurements."
tags: [merge-queue, ci, github-actions, mergify, ai-agents]
---

## Summary

A merge queue is a bot that merges approved pull requests into a busy branch in order, after testing each one on top of the latest base branch plus the PRs ahead of it. It solves "two green PRs make a red main": a PR that passed CI only proved it works against the main it was tested on. GitHub ships one per protected branch; Mergify, Graphite, Trunk and Aviator sell richer ones.

## The problem it solves

- PR A renames `getUser()` to `fetchUser()`, PR B adds a caller of `getUser()`. Each passes CI against the old main. Merge both and main is broken; nobody ever tested A + B together.
- The manual fix is GitHub's "Require branches to be up to date before merging" (strict status checks). It is safe but every merge makes every other open PR stale, so developers rebase, rerun CI, and race each other. GitHub says the merge queue "provides the same benefits" but "does not require a pull request author to update their pull request branch".
- A queue is not a CI saver by itself: it adds a CI run per merge group. The savings come from tiering (below).

## How it works

GitHub's queue, as documented:

1. A PR passes the branch protection checks; a user with write access adds it to the queue. PRs merge first-in, first-out.
2. The queue "creates temporary branches with a special prefix" (`gh-readonly-queue/{base_branch}` for third-party CI) that combine the latest base branch with the changes from the PRs ahead.
3. Each group dispatches a `merge_group` webhook of type `checks_requested`. GitHub Actions workflows must add `on: merge_group` next to `pull_request`, otherwise "the merge will fail as the required status check will not be reported". `merge_group` "is separate from the `pull_request` and `push` events".
4. When a group's required checks pass, GitHub merges it with the configured merge method (merge, rebase, squash).
5. A PR is removed from the queue when CI reports a failure, when CI does not respond within the status-check timeout, when a user removes it, or when a branch-protection failure cannot be resolved automatically; the entries behind it are rebuilt without it. "Jumping the queue" is allowed but forces a rebuild of every in-progress PR.
6. Knobs: build concurrency (1 to 100 concurrent `merge_group` webhooks), minimum and maximum PRs per merge (1 to 100) with a wait time, status-check timeout, and "only merge non-failing pull requests".

Terms the vendors use (Mergify docs):

- **Speculative / parallel checks**: "test queued pull requests ahead of time within a single ordered queue" by stacking main+1, main+1+2, main+1+2+3 and running CI at once; `max_parallel_checks` caps how many run.
- **Batches**: several PRs tested in one CI run (`batch_size`); when a batch fails Mergify splits it and retests the parts to isolate the culprit, which is then removed.
- **Scopes** (what the draft chat called partitions): the areas of the codebase a PR touches, computed from file patterns or uploaded by Bazel/Nx/Turborepo. In parallel mode "each scope is a lane"; a failure in one scope "does not block unrelated scope queues". A PR with two scopes costs one batch and one CI run, tested on top of the work ahead in both lanes.

Tiered CI for many parallel AI agents, the setup that actually cuts minutes:

| Tier | Trigger | Run here |
|---|---|---|
| PR checks | `pull_request`, every push | lint, typecheck, unit tests on affected packages, preview deploy, review bots |
| Queue checks | `merge_group`, once per group | full build, integration / e2e, migrations against a real DB; keep typecheck here too, it catches the `getUser` break |
| After merge | `push` to main | release build, deploy; no retest, the queue tested this exact commit |

A job gated with `if: github.event_name == 'merge_group'` "reports Success" when skipped on the PR, so it can be a required check that only truly runs in the queue. Do not gate required jobs with `paths:` filters instead: a workflow skipped by path filtering stays "Pending" and blocks merging.

Agent-specific cautions: queue only green PRs (each in-queue failure cancels the speculative runs behind it); flaky tests kick good PRs out; the queue cannot resolve text conflicts, so keep agent PRs small; send queue failures back to the agent automatically.

## When to use it, when to skip it

| Use it when | Skip it when |
|---|---|
| Many PRs per day on one branch (GitHub: "particularly useful on branches that have a relatively high number of pull requests merging each day") | A few PRs a day and fast CI; the strict "up to date" rule is enough |
| Many agents or developers in a monorepo, slow CI, stacked PRs | CI is not split into tiers yet; the queue alone adds runs |
| You want "click queue, walk away" instead of babysitting rebases | Required checks are skipped by path filters (they would block the queue) |

Order that worked in the draft chat: tiered CI, then affected-only builds with caching, then the queue, then scopes once one shared line is the bottleneck.

## Diagram

[diagrams/merge-queue.html](../../diagrams/merge-queue.html): the two-green-PRs failure, the manual rebase race, the queue's speculative temporary branches with one PR kicked out, the three CI tiers, scope lanes, and the illustrative CI-minute comparison. It does not show batching, priorities, or the GitHub settings UI.

## Sources

- https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/configuring-pull-request-merges/managing-a-merge-queue: backs FIFO merging, temporary branches and the `gh-readonly-queue/` prefix, the `merge_group` / `checks_requested` webhook, build concurrency and merge-limit ranges, removal causes, and jumping the queue.
- https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches: backs "same benefits as up to date without the author updating", the "high number of pull requests" sentence, and strict vs loose status checks.
- https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows: backs the `merge_group` event, its single `checks_requested` type, and the failure when the trigger is missing.
- https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/collaborating-on-repositories-with-code-quality-features/troubleshooting-required-status-checks: backs conditional-skipped jobs reporting Success and path-filtered workflows staying Pending.
- https://docs.mergify.com/merge-queue/batches/: backs `batch_size`, batch splitting to isolate the culprit, and removal of the failing PR.
- https://docs.mergify.com/merge-queue/queue-modes: backs parallel checks as speculative CI, `max_parallel_checks`, and failures not blocking unrelated scope queues.
- https://docs.mergify.com/merge-queue/scopes: backs scopes from file patterns or Bazel/Nx/Turborepo uploads, "each scope is a lane", and a two-scope PR costing one CI run.
- https://docs.mergify.com/merge-queue/speculative-checks/: backs stacking PR 1, PR 1+2, PR 1+2+3 and removing the culprit when a batch fails.

---
title: Redesigning source control at Anthropic (Scion)
kind: article
area: vcs
date: 2026-10-21
status: verified
diagram: diagrams/anthropic-source-control-redesign.html
sources:
  - https://www.anthropic.com/engineering/redesigning-source-control
unverified:
tags: [scion, anthropic, monorepo, git, ai-agents, source-control]
---

## What the article claims

- Coding is no longer the bottleneck; developer infrastructure sized for humans (source control, builds, CI) is. "In May 2026, Anthropic averaged around 2,500 code changes merged per day"; "as of September, we merged over 30,000 per day."
- The main monorepo's history "alone is almost 100 GB" and "a compressed working copy is over 25 GB". "We perform on the order of 25 million code checkouts per day for CI"; "Fetched naively, that would be over 600 PB of git traffic per day." At peak they "operated several hundred internal git mirrors", which got harder to run as merge rates grew.
- They built **Scion**, an in-house version control system, plus **Antique** (code review), code search, a source-control IAM system, and new ways to land large volumes of reviewed code. The monorepo was "fully migrated ... from git in five months"; "A team of seven people built both Scion and Antique" while daily merged volume grew 10x.
- Design principles: clients are always online (git's offline-first assumption does not apply); client-server asymmetry (server owns history, blame, commits, merges, rebases; client keeps a bounded cache and working copy); source control exposed via API so jobs such as automated bisect and revert need no local repo; everything horizontally scalable with no full repo copy and no packfile maintenance; commits are cheap, incremental and lock-free, with refs discouraged except main and release branches; the working copy is optional and sparse, with "virtualized checkouts, which materialize files on demand".
- Data model: the unit of work is a "changeset" in a write-ahead log, storing the files changed relative to the first parent with content-addressed pointers to blobs. "Derived data" (blame, path history, commit-graph skip lists) is computed recursively from parent values.
- Fetch protocol: the `scn` client "operates exclusively on trees and blobs the client wants"; "Starting from a root tree, the client checks its local store for missing trees and requests them recursively." It is resumable, and sparse checkouts "only contain the directories the client asks for".
- Architecture: three tiers, scion-daemon (per node), scion-cache (regional), scion-server (authoritative, backed by Spanner and cloud storage). Strict tiering isolates failure domains.
- Landing code: "We haven't been able to effectively replicate the industry standard of a global repository lock on the merge queue," so they invested in post-merge and pre-deploy testing instead.
- Prior art: "notably Google's Piper and Meta's Sapling"; Mononoke, Sapling's server, "pioneered many of the techniques we use in Scion".
- Security: Scion "supports rich access control on repositories, as well as the ability to apply permissions to individual directories"; Antique "can express complex semantic constraints on approvals" to "differentiate simple things like comment-only changes from sensitive logical changes".
- Working with Claude: an early Claude rewrite of the git-to-Scion ingestion pipeline "consumed 20x less memory and was 4x faster". "Very often, we found that the role of the engineers was to tell Claude 'no.'" Some agents "tried to brute force looking up when a change merged to main by linearly reading the commit log." Claude built hermetic simulations, fuzzing and rate limiting. "Superfan" Claude agents each watch one component's telemetry and logs.
- What they hope git platforms borrow: server-side RPCs like Scion's, possibly standardised across forges; new server architectures; a different negotiation protocol; sparse and shallow features that compose. They want to partner with GitHub, GitLab and the OSS community.

## Key numbers

| Number | Quote from the article |
|---|---|
| 2,500 to 30,000+ merges/day (May to Sep 2026, about 12x) | "around 2,500 code changes merged per day" ... "over 30,000 per day" |
| ~100 GB history, 25 GB+ working copy | "almost 100 GB" / "over 25 GB" |
| ~25 million CI checkouts/day, 600+ PB/day if naive | "on the order of 25 million code checkouts per day" / "over 600 PB of git traffic per day" |
| Several hundred git mirrors at peak | "we operated several hundred internal git mirrors" |
| Five months, seven people | "fully migrated our monorepo from git in five months" / "A team of seven people built both Scion and Antique" |
| 200x smaller footprint | "200x smaller than our previous git mirroring infrastructure" |
| >99% cache hit rate per tier | "cache hit rate of over 99%" |
| 25% CI CPU capacity win | "CI alone realized a 25% CPU capacity win" |
| Checkouts in seconds, not minutes | CI and research checkouts take seconds instead of minutes |
| 20x / 4x | ingestion pipeline rewrite "consumed 20x less memory and was 4x faster" |

## What it leaves out

- How conflicts between concurrent lock-free changesets are detected and repaired after landing, and how often post-merge tests fail.
- What the changeset and derived-data schemas look like in Spanner, and how derived data is invalidated.
- Any latency or cost numbers for the Spanner-backed server itself, beyond the tier hit rates.
- Whether the fetch protocol or RPC surface will be published as a spec.

## Status

In production inside Anthropic, not released. The article: "Scion is by no means a suitable replacement for git for general use." "Practically speaking, it's difficult to offer Scion in a form that would be broadly useful to other organizations." Adopters "would need to migrate every piece of the development lifecycle" because Scion "requires its own code review, code search, and CI", and "Our internal versions of these components are highly bespoke, so they would be difficult for other organizations to adopt." The post offers no download, product, or open-source release; it shares design ideas for git platforms to borrow. Published October 13, 2026; "Updated October 15 to include more detail on the projects that pioneered many of the techniques we use in Scion." Meta's Sapling is the open-source system it points to as prior art.

## My takeaways

- The whole design follows from one assumption flip: clients are always online, so history and computation move to the server and clients fetch only what they ask for.
- It is a monorepo-at-agent-scale tool. For normal repos the useful part is the wishlist: server-side RPCs, composable sparse and shallow checkouts.
- Their abandoning a global merge-queue lock is the opposite of the merge-queue note; both are right at their own merge rates.

## Diagram

[diagrams/anthropic-source-control-redesign.html](../../diagrams/anthropic-source-control-redesign.html): the growth numbers, git's thick clients vs Scion's thin always-online clients, the tree-and-blob fetch walk, the three cache tiers over Spanner, and the results table. It does not cover Antique's approval rules, the security model, or the Claude-written tooling.

## Sources

- https://www.anthropic.com/engineering/redesigning-source-control: backs every claim, number, quote and the release status above.

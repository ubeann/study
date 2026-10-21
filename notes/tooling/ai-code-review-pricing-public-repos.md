---
title: Are CodeRabbit and Macroscope free for public repos?
kind: lookup
area: tooling
date: 2026-10-21
status: verified
sources:
  - https://www.coderabbit.ai/pricing
  - https://docs.coderabbit.ai/management/plans
  - https://macroscope.com/open-source
  - https://macroscope.com/blog/free-for-open-source
  - https://docs.macroscope.com/pricing
  - https://macroscope.com/pricing.md
  - https://docs.macroscope.com
unverified:
tags: [coderabbit, macroscope, code-review, pricing, open-source]
---

## Question

Can I run CodeRabbit on a public repo without paying? Is Macroscope also free for public repos?

## Answer

CodeRabbit: yes, any public repository gets free reviews with Team-plan features. Macroscope: only by application, and only for non-commercial open-source projects; a public repo on its own does not qualify.

## Evidence

**CodeRabbit**

> receive free reviews forever for public repositories

https://www.coderabbit.ai/pricing, checked 2026-10-21. Paid plans on the same page: Essentials $24 per user per month billed annually ($30 month-to-month), Team $48 annually ($60 monthly), Advanced $72, Enterprise "Contact Us". The FAQ adds: "Each plan does include a per-developer hourly review allowance to manage bursts of activity."

> Unlimited public repositories, no credit card required.

> Open-source projects receive Team features with no paid subscription required.

> OSS reviews use a separate rate-limit tier that varies with the project's community and popularity

https://docs.coderabbit.ai/management/plans, checked 2026-10-21. OSS limits listed there: 1 to 10 PR reviews per developer per hour (by star count), 100 to 300 files per review, 25 chat messages per hour. The Free plan for private repos is "PR summarization only; code reviews are available through the VS Code extension and CLI." The docs call the entry paid tier "Essentials plan (previously called Pro)", which resolves the old chat's Pro vs Pro+ confusion.

**Macroscope**

> We're offering free Macroscope credits for qualified non-commercial open source projects.

> Your project must use an open source license (MIT, Apache 2.0, GPL, etc.).

> Your project must be used for non-commercial purposes and not sold or offered as a paid product or service.

> we reserve the right to limit usage based on reasonable use criteria.

https://macroscope.com/open-source, checked 2026-10-21. Access is by an application form ("Apply for free access"); projects start on Budget mode and can move to Balanced or Precise, Ultra is paid only. The announcement post, https://macroscope.com/blog/free-for-open-source (Nov 4, 2025), says the same and gives no numeric cap.

> Macroscope charges for usage, with no seat fees.

> New workspaces receive $100 in free usage.

> Each workspace also gets 1,000 free Agent credits per month; these credits do not roll over.

https://docs.macroscope.com/pricing, checked 2026-10-21. Code review is billed per KB of diff: Budget $0.025, Balanced (default) $0.05, Precise $0.06, Ultra $0.20 per KB, "10 KB minimum per review". https://macroscope.com/pricing.md, same day: "A seat-based plan ($30/active developer/month) still applies to customers who subscribed before March 2026."

## Caveats

Plan names, prices, and limits change. Macroscope's pages state no policy for public repositories as such: eligibility is license plus non-commercial use plus "community fit", judged on an application, and no page fetched here gives a numeric cap, a cap tied to developer count, or what happens when it is hit. The old chat's "$30 per developer, five-seat minimum" and "$0.05/KB" were both partly right: $30/seat is the legacy plan, $0.05/KB is the current Balanced rate, and no seat minimum appears on any vendor page. Macroscope's docs mention only GitHub pull requests and the CLI; they do not say GitLab is unsupported. CodeRabbit's public-repo tier is rate-limited by project popularity, so a small new repo sits at the low end of 1 to 10 reviews per developer per hour.

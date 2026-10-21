---
title: Is Cloudflare D1 built on MySQL or PostgreSQL?
kind: lookup
area: databases
date: 2026-10-21
status: verified
sources:
  - https://developers.cloudflare.com/d1/
  - https://developers.cloudflare.com/d1/platform/limits/
  - https://developers.cloudflare.com/d1/reference/time-travel/
  - https://developers.cloudflare.com/d1/best-practices/read-replication/
  - https://developers.cloudflare.com/hyperdrive/
  - https://neon.com/docs/introduction
  - https://neon.com/docs/introduction/architecture-overview
unverified:
tags: [cloudflare, d1, sqlite, neon, postgres, hyperdrive]
---

## Question

Is Cloudflare D1 built on MySQL or PostgreSQL, the way Neon is built on Postgres?

## Answer

Neither. D1 is a serverless database with SQLite's SQL semantics; Neon runs standard Postgres. If you need Postgres or MySQL from a Worker, Cloudflare's answer is Hyperdrive, not D1.

## Evidence

> D1 is Cloudflare's managed, serverless database with SQLite's SQL semantics

> Execute SQL with SQLite's SQL compatibility and D1 Client API.

https://developers.cloudflare.com/d1/, checked 2026-10-21. The same page says D1 offers "Worker and HTTP API access" and lets you "create thousands of databases at no extra cost for isolation".

> Maximum database size | 10 GB (Workers Paid) / 500 MB (Free)

> Note that the 10 GB limit of a D1 database cannot be further increased.

https://developers.cloudflare.com/d1/platform/limits/, checked 2026-10-21. Same page: "Databases per account | 50,000 (Workers Paid) / 10 (Free)", "Time Travel duration (point-in-time recovery) | 30 days (Workers Paid) / 7 days (Free)".

> Time Travel is D1's approach to backups and point-in-time-recovery.

https://developers.cloudflare.com/d1/reference/time-travel/, checked 2026-10-21 ("up to 30 days in the past (Workers Paid plan) or 7 days (Workers Free plan)").

> To use read replication, you must use the D1 Sessions API

> Sessions API ensures sequential consistency for all queries in a session.

https://developers.cloudflare.com/d1/best-practices/read-replication/, checked 2026-10-21.

> Hyperdrive supports any Postgres or MySQL database

https://developers.cloudflare.com/hyperdrive/, checked 2026-10-21.

> Lakebase Postgres is a serverless database that splits the system into two independent layers: compute and storage.

> The compute layer is where Postgres actually runs.

https://neon.com/docs/introduction/architecture-overview, checked 2026-10-21. The docs home (https://neon.com/docs/introduction) describes Neon as "Serverless Postgres with branching, autoscaling, scale to zero, and instant restore."

## Caveats

Plan names, prices, and limits change. The 10 GB / 500 MB cap is per database and the limits page says it cannot be raised; the intended pattern is many small databases (50,000 per paid account). The D1 overview does not name the Sessions API; that comes from the read-replication page, which gives no GA-or-beta status. Neon's docs now brand the engine "Lakebase Postgres"; the comparison here is engine-level (SQLite vs Postgres), not feature-level. The old chat's claim that each D1 database runs inside a Durable Object is not on any page fetched here and is left out.

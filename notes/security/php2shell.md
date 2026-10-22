---
title: "PHP2Shell: CVE-2026-109618 and CVE-2026-109619"
kind: concept
area: security
date: 2026-10-22
status: draft
diagram: diagrams/php2shell.html
sources:
  - https://www.php.net/archive/2026.php
  - https://github.com/php/php-src/security/advisories/GHSA-w8hw-w9fx-j79w
  - https://github.com/php/php-src/security/advisories/GHSA-6wmf-4rv5-p3rp
  - https://thephp.foundation/blog/2026/10/21/todays-php-security-release-and-how-were-strengthening-php-security/
  - https://calif.io/php2shell
  - https://www.wiz.io/blog/php2shell-php-rce
  - https://aws.amazon.com/blogs/security/amazon-threat-intelligence-identifies-php2shell-exploitation-prior-to-public-disclosure-in-madpot/
  - https://www.ncsc.gov.uk/news/critical-security-issue-affecting-php
  - https://cert.europa.eu/publications/security-advisories/2026-018/
  - https://watchtowr.com/intelligence/php-cve-2026-109618-faq/
  - https://watchtowr.com/intelligence/php-remote-code-execution-php2shell/
  - https://www.securityweek.com/php2shell-mass-exploitation-looms-for-leaked-critical-php-flaws/
  - https://www.securityweek.com/php2shell-patches-land-as-first-exploitation-attempts-hit-sensors/
  - https://www.huntress.com/blog/php2shell-leaked-critical-php-vulnerabilities
  - https://www.bleepingcomputer.com/news/security/php-ships-emergency-patches-for-critical-php2shell-flaws/
  - https://www.helpnetsecurity.com/2026/10/21/php2shell-php-vulnerabilities-patched-cve-2026-109618-cve-2026-109619/
  - https://www.insurancebusinessmag.com/us/news/cyber/nsa-warns-of-critical-website-flaw-as-wikipedia-european-government-websites-taken-offline-594432.aspx
  - https://en.wikipedia.org/wiki/PHP2Shell
unverified:
  - "The NSA advisory itself (nsa.gov article 4630238) returned HTTP 403; its wording is quoted from Insurance Business coverage and Wikipedia."
  - "Reuters and CNBC pieces naming the exact government sites (Overheid.nl, Valtori, French prefectures, Kombit, 4chan, Something Awful) could not be fetched; those names come from Wikipedia and a web-search summary. The country list itself is confirmed by Insurance Business."
  - "Wikipedia says the GitHub advisories went live at 13:34 UTC, 26 minutes early. The advisory pages show only the date, not the time."
  - "Wikipedia says Calif used GPT-5.6 Sol. Calif's own page does not name a model and says no non-public models or special harness were used in initial discovery."
  - "Wikipedia and the Help Net Security summary say watchTowr rated the bugs 10.0. The watchTowr FAQ fetched today shows 9.2; the PHP Foundation says the leaked material carried a wrong 10.0 score."
  - "VirusTotal's figure of 5.1 million exposed PHP hosts is quoted via BleepingComputer, not from VirusTotal."
  - "CISA's statement that it had not observed exploitation as of 21 October comes from Wikipedia only; no CISA page was fetched."
tags: [php, cve, rce, memory-safety, incident-response, coordinated-disclosure, use-after-free, ai-vulnerability-research]
---

## Summary

PHP2Shell is the name given to two critical PHP engine bugs, CVE-2026-109618 ("RCE via OOM corruption") and CVE-2026-109619 ("RCE via stale global state"), each rated CVSS v4.0 9.2, that let an unauthenticated HTTP request execute code as the web worker before any application code runs. They live in PHP itself, not in WordPress or any framework, and affect every release back to PHP 5.4. Fixes shipped in PHP 8.5.12, 8.4.27, 8.3.36 and 8.2.35 on Wednesday 21 October 2026 at 14:00 UTC, one day earlier than planned, after the embargo leaked.

## The problem it solves

This is an incident note rather than a tool note, so "the problem" is the one the attackers exploit and the one the maintainers had to manage.

**The technical problem.** PHP handles a fatal out-of-memory error by `longjmp()`-ing out of whatever it was doing. Per GHSA-w8hw-w9fx-j79w, "when an allocation fails while request memory is partly initialized, the request shutdown can process invalid state." Per GHSA-6wmf-4rv5-p3rp, PHP also keeps global state (error, session, object and extension state) that "a persistent worker can retain after the request's memory is freed", and some operations run before normal request initialization and read that stale state. Both are CWE-416 use-after-free at heart.

**The history problem.** Calif's writeup traces the pattern back to Stefan Esser's 2004 memory-limit exploit, a 2020 report (bug #78876) about multipart out-of-memory side effects, and a 2024 OSTIF and Quarkslab audit that found a related OOM issue "but exploitation was not proven, so it was not fixed." The PHP Foundation's own post admits the OOM fix "stalled for months" and calls that "a structural failure" caused by the security team's decision process and a lack of resources.

**The scale problem.** Wiz puts PHP at about 70% of websites, and since the bugs are in the engine "even a simple 'Hello World' app on an affected version can be vulnerable." BleepingComputer quotes VirusTotal estimating over 5.1 million exposed PHP hosts. Many run end-of-life versions bundled in appliances, cPanel or Plesk, where nobody can ship a backport.

## How it works

### How it was found

- **1 to 21 August 2026, Calif.** Ryan Austin of Calif, invited to OpenAI's "Patch the Planet" initiative, chose PHP as "a high-impact target." The team used LLM "audit buddy" agents for source review and deliberately prompted them from PHP's older known primitives rather than only recent bugs. Calif "surfaced the first OOM-fatal and an interruption point" on 1 August, reported the first bug to PHP on 11 August, shared a working RCE proof of concept on 14 August (PHP confirmed it on 15 August) and reported a second bug on 21 August. Calif states: "No special harness, or access to models that weren't generally available to everyone were used in the initial bug discovery."
- **19 to 24 August 2026, Wiz.** Wiz Research independently found its first PHP bug on 19 August using Wiz Atlas, Gemini 4 Argon and GPT-5.6 Sol, confirmed unauthenticated RCE against Drupal on 20 August and WordPress on 21 August, and reported to PHP on 24 August. Wiz says the models "contributed to finding the vulnerability, exploit development, attack detection, and patch validation."
- **Credits.** Both GitHub advisories credit reporters lcfr-hax, sagitzadik-wiz, tomash-areion and amyb-oai (OpenAI), and remediation developers arnaud-lb, iluuu1994, bukka and ndossche.
- **21 September 2026.** PHP acknowledged the reports and began a coordinated fix with distributors and LTS vendors under embargo. The release was scheduled for 22 October 2026 at 14:00 UTC.

### Mechanics of CVE-2026-109618, "RCE via OOM corruption"

From Calif's writeup and the GitHub advisory, in attack order:

1. **Create memory pressure without tripping `post_max_size`.** PHP checks `post_max_size` against the declared `Content-Length`, not bytes actually read. A chunked HTTP/1.1 body, a length-less HTTP/2 body, or input decompression gets around it.
2. **Exhaust `memory_limit`.** A multipart file part with `filename=""` carries no data and does not count toward `max_file_uploads`, yet PHP still creates six `$_FILES` entries for it. With deeply nested field names such as `files[a][b][c]...`, hundreds of these records blow past the default 128 MiB heap while staying under the default 8 MiB `post_max_size`. The attacker chooses which later allocation fails.
3. **Interrupt an allocation at a useful point.** In `_zend_hash_str_add_or_update_i`, PHP increments `nNumUsed` and `nNumOfElements` before the new bucket's key is allocated. If that allocation fatals, the key assignments are skipped, but teardown still treats the bucket as live.
4. **Reach code execution.** On Apache mod_php the registered request cleanup still runs even though startup failed, so it destroys the malformed table and interprets stale bytes as a live bucket. The advisory says the reporters "validated this over HTTP against mod_php with default limits and prefork configurations." PHP-FPM exits its worker on startup failure, so this exact path does not work there; FPM is "susceptible in principle" but no complete FPM exploit was demonstrated for this CVE.

Calif lists pre-auth RCE confirmed against WordPress 7.1.2, Drupal 11.4.3, Joomla 6.1.4, MediaWiki 1.46.2, Roundcube 1.7.4, Magento 2.4.9, phpMyAdmin 5.2.3, Nextcloud 35.0.1, Laravel 13.11, Symfony 8.1 and WHMCS.

### Mechanics of CVE-2026-109619, "RCE via stale global state"

This one needs two requests to the same worker, a "producer" and a "consumer":

1. **Producer.** A multipart request carrying the `PHP_SESSION_UPLOAD_PROGRESS` field makes the Session module decode upload-progress data before module initialization has finished. An out-of-memory fatal during input processing destroys the request heap but leaves `PS(id)` and the unserializer's bookkeeping in `BG(unserialize)` pointing at freed memory.
2. **Consumer.** The next request on that worker reaches cleanup, or an early INI diagnostic, before initialization resets those pointers. It reads attacker-reclaimed memory as live state.
3. **Outcome.** A heap use-after-free. Arranged carefully, RCE as the web worker; alternatively, Wiz showed PHP will URL-encode the reclaimed memory into a `PHPSESSID` response cookie, leaking passwords, tokens and keys "even if the application never calls `session_start()`." This was validated against both mod_php and PHP-FPM.
4. **Extra producer on PHP 8.5.** The new `fatal_error_backtraces` feature stores the last fatal backtrace in `EG(last_fatal_error_backtrace)`; an OOM leaves it dangling. The advisory calls this "the strongest confirmed case" with confirmed RCE against mod_php.

Wiz also describes an OPcache type confusion where an allocation failure between advancing a per-process frontier and publishing the shared one leaves slots inconsistent across workers.

### The fix

The patch "replaces the `longjmp()` path with a soft OOM at the next VM safe point and a hard OOM via `_exit()` at a higher threshold", so no operation is abandoned half-done. The advisory also recommends keeping `session.upload_progress.enabled=0` even after patching if the feature is not needed.

### Timeline of the incident (all times UTC)

| When | What |
|---|---|
| 17 Oct | Amazon's MadPot honeypot records a PHP2Shell exploitation attempt from "an IP address associated with a commercial anonymization network", four days before disclosure. Amazon: "it remains unclear how the actor acquired exploit information prior to disclosure." |
| 20 Oct ~15:00 | The two CVE IDs start circulating on forums and social media (a Hacker News post, later removed, and an X post). The leaked material carried a wrong release time and a wrong CVSS 10.0. PHP Foundation: "No technical details leaked." |
| 20 Oct 19:30 | PHP pre-announces "an upcoming security release" for 21 October at 14:00 UTC, pulling the date in by one day, and says: "We reserve the right to release earlier should details of either vulnerability become public, or should evidence of active exploitation appear in the wild." watchTowr publishes an FAQ at 19:35. |
| 20 Oct ~20:00 | Calif mistakenly publishes a demo video of its proof of concept against several targets, revealing "a few additional, though limited, details." |
| 20 Oct ~23:00 | Mitigation advice sent to some critical infrastructure providers "may have leaked as well." |
| 21 Oct ~02:00 to 08:00 | Earliest proof-of-concept exploits appear; Amazon sees more honeypot attempts from varied anonymized infrastructure, "suggesting multiple actors." Hadrian reports exploitation "at scale" a few hours before the fix, mostly via the session upload-progress path. |
| 21 Oct 07:41 | Chainguard publishes built images containing the fix, more than six hours early. PHP Foundation: "Any attacker could now study the fix in binary form while every other PHP user was still waiting for the release." |
| 21 Oct ~05:00 | Cloudflare deploys WAF rules "nine hours before public disclosure" and reports early attacks on both CVEs. |
| 21 Oct 14:00 | PHP 8.5.12, 8.4.27, 8.3.36 and 8.2.35 ship with both GitHub advisories. "There are no complete mitigations, updating is the only safe choice." Minimal source backports for 7.4, 8.0 and 8.1. |
| 21 Oct 15:20 | CERT-EU publishes advisory 2026-018. UK NCSC publishes its alert the same day. |
| 21 Oct, during the day | Government agencies in the Netherlands, Finland, France, Denmark and Germany, plus Wikimedia, the Internet Archive and others, take sites offline as a precaution. watchTowr's honeypots see first attempts; "nobody has been successful to date." |
| 21 Oct ~20:30 | Wiz and Calif publish full technical writeups. PHP Foundation publishes its post-mortem. |

### Decisions, by actor

**PHP maintainers and the PHP Foundation.**
- Faced with the leak two days out, they chose a one-day pull-in rather than an immediate release because "distributors were still packaging the fix, so an immediate release would have left many users without an easy update path." They call that "the best compromise" between speed, preparation time and a simple update path.
- They shipped fixes for the four supported branches, source-only patches for 7.4, 8.0 and 8.1, and nothing for older. "PHP 8.1 and older no longer receive updates from the PHP project."
- They published mitigations but refused to oversell them: the measures "reduce exposure but do not make an installation safe." The mitigations are `memory_limit=-1` enforced via `php_admin_value`, a low `post_max_size` (8M, or 32M with uploads), container or cgroup limits that kill only the worker, `session.upload_progress.enabled=0` with every worker restarted, and on 8.5 `fatal_error_backtraces=0`. They explicitly warn against `RLIMIT_AS` or `RLIMIT_DATA`, which can trigger the vulnerable allocation failure, and say that hiding diagnostics with `error_reporting`, `display_errors` or `@` does not help.
- They warned users of the volunteer-run `rpms.remirepo.net` to switch to distribution packages or build from source because that repository "will not have updates today."

**Governments and national CERTs.**
- UK NCSC: find PHP across the estate including vendor appliances, update immediately, and "if mitigations aren't possible or would take too long, consider taking the system offline until it can be fully mitigated." Before upgrading an exposed system, "assess the system for any evidence of compromise" and "look for backdoors installed by an attacker."
- US NSA: where an immediate update is impossible, recommend "taking the system offline until the problem is fully fixed." (quoted via press coverage, see unverified)
- CERT-EU: patch immediately; those who cannot should "consider isolating internet-facing resources from public networks until the fix is applied and verified." Temporary mitigations "do not make an installation safe."
- Acted on it: Dutch, Finnish, French, Danish and German agencies took public sites down for the day. Japan's national cybersecurity office advised immediate updates.

**Businesses, hosts and large sites.**
- Cloudflare pushed WAF rules to all plans including free, and its CTO called them temporary.
- Wikimedia, the Internet Archive, 4chan and several Japanese hosts and commerce sites chose downtime over exposure.
- Wiz coordinated ahead of time with Google Cloud, Cloudflare, AWS, Microsoft, Automattic, Fastly, Akamai, Imperva and Vercel so mitigations existed at disclosure.
- Detection guidance from Huntress and Amazon: alert on PHP workers (php-fpm, php-cgi, httpd, apache2) spawning shells, interpreters or download tools; look for POST bodies containing `PHP_SESSION_UPLOAD_PROGRESS`; look for repeated 500s after memory-limit errors and oversized `PHPSESSID` cookies; start incident timelines from 17 October, not from disclosure day.

### Post-mortem

What the PHP Foundation says it is changing:

- **Decision process.** The security team now decides by majority with compromises encouraged, and leadership can "help steer the situation to a timely resolution" when a complex fix stalls, as the OOM fix did for months.
- **People.** An Ecosystem Security Team was created in May 2026; Alexandre Daubois joined in July for triage and fixes, Daniel Scherzer in September for hardening, and Volker Dusch serves as AI Security Engineer in Residence.
- **Tooling.** Sanitizer builds, reproduction environments matching CI on every supported platform, and OpenAI's Daybreak Red for validating reports and exploits, added since an earlier Claude Mythos scan.
- **Funding.** A direct ask to companies, hosts, distributions and governments to "sponsor the PHP Foundation at a level that reflects how much you rely on it", and for sustained public funding.

What the incident teaches more generally:

- A bug that "cannot be proven exploitable" is not the same as a safe bug. The 2024 audit finding sat for two years until LLM-assisted researchers connected it to twenty-year-old primitives.
- Coordinated disclosure has many weak points: forum chatter, an accidental video, mitigation memos to critical infrastructure, and a downstream vendor publishing early. The Foundation says the process "worked well until 2 days ahead of the targeted release date."
- Someone was probing honeypots on 17 October, before any public leak. Amazon's advice is to measure patch timelines "in hours rather than days" for critical internet-facing bugs.
- Taking a site offline is a legitimate security control. Five European governments and the Wikimedia Foundation used it.

## When to use it, when to skip it

Read as "when to treat a PHP bug like this one":

| Treat as an emergency when | Lower urgency when |
|---|---|
| The bug is in the engine and reachable before application code runs (mod_php, FPM, cgi all listed) | The bug needs a specific extension or application path you do not use |
| The vendor says no configuration change makes you safe | A documented ini or config setting fully closes the path |
| Details or PoCs have leaked, or honeypots report attempts | Embargo intact and no exploitation evidence |
| You run end-of-life PHP with no vendor backports | You are on a supported branch with distro packages ready |

## Diagram

[diagrams/php2shell.html](../../diagrams/php2shell.html): five panels telling the story in order: what happened, how it was found (2004 to 2026), the two exploit chains step by step, a timeline from 17 to 22 October with the decisions each actor made, and a post-mortem table. It does not show the exact heap layout for the use-after-free, the OPcache type-confusion variant, or per-distribution package versions.

## Sources

- https://www.php.net/archive/2026.php: 20 Oct pre-announcement text and time, 21 Oct advisory, fixed versions, "no complete mitigations", all mitigation settings, remirepo warning, EOL statements, mod_php risk.
- https://github.com/php/php-src/security/advisories/GHSA-w8hw-w9fx-j79w: CVE-2026-109618 summary, CVSS vector and 9.2, CWEs, longjmp root cause, mod_php validation, FPM susceptibility, patch design, credits.
- https://github.com/php/php-src/security/advisories/GHSA-6wmf-4rv5-p3rp: CVE-2026-109619 summary, producer and consumer model, session upload progress and fatal_error_backtraces paths, mitigations, credits.
- https://thephp.foundation/blog/2026/10/21/todays-php-security-release-and-how-were-strengthening-php-security/: leak timeline (15:00, 19:30, 20:00, 23:00 UTC), original 22 Oct date, Chainguard 07:41 UTC, "structural failure", process, people, tooling and funding changes.
- https://calif.io/php2shell: Calif discovery dates, Patch the Planet, LLM methodology, 2004/2020/2024 history, five-step mechanics, affected application list, no-special-models statement.
- https://www.wiz.io/blog/php2shell-php-rce: Wiz dates (19, 20, 21, 24 Aug, 21 Sept), tools used, 70% figure, memory disclosure via PHPSESSID, OPcache variant, partner coordination list, hunting indicators.
- https://aws.amazon.com/blogs/security/amazon-threat-intelligence-identifies-php2shell-exploitation-prior-to-public-disclosure-in-madpot/: 17 Oct honeypot attempt, anonymization network, 02:00 to 08:00 UTC attempts, PHP_SESSION_UPLOAD_PROGRESS indicator, "hours rather than days".
- https://www.ncsc.gov.uk/news/critical-security-issue-affecting-php: NCSC priority actions, offline advice, backdoor check wording.
- https://cert.europa.eu/publications/security-advisories/2026-018/: advisory number, 15:20:18 publication time, isolation wording, affected products.
- https://watchtowr.com/intelligence/php-cve-2026-109618-faq/: FAQ publish time 19:35 UTC 20 Oct, 9.2 score, honeypot observation on 21 Oct.
- https://watchtowr.com/intelligence/php-remote-code-execution-php2shell/: list of affected SAPIs, "ini changes ineffective" position.
- https://www.securityweek.com/php2shell-mass-exploitation-looms-for-leaked-critical-php-flaws/: 20 Oct warning, scheduled 14:00 UTC, Sansec 15-minute prediction, Benjamin Harris quote.
- https://www.securityweek.com/php2shell-patches-land-as-first-exploitation-attempts-hit-sensors/: Hadrian at-scale report, Cloudflare nine hours early, Chainguard quote, Jake Knott quote.
- https://www.huntress.com/blog/php2shell-leaked-critical-php-vulnerabilities: Hacker News post origin, detection guidance, ~20:30 UTC writeup timing.
- https://www.bleepingcomputer.com/news/security/php-ships-emergency-patches-for-critical-php2shell-flaws/: Cloudflare CTO statement, VirusTotal 5.1 million figure, Gina Peter Banyard statement, discoverer list.
- https://www.helpnetsecurity.com/2026/10/21/php2shell-php-vulnerabilities-patched-cve-2026-109618-cve-2026-109619/: "emergency security updates", Cloudflare free-plan rules, remirepo note.
- https://www.insurancebusinessmag.com/us/news/cyber/nsa-warns-of-critical-website-flaw-as-wikipedia-european-government-websites-taken-offline-594432.aspx: NSA offline wording, country list (NL, FI, FR, DK, DE), Wikipedia partially offline.
- https://en.wikipedia.org/wiki/PHP2Shell: aggregate timeline, named sites taken offline, Japan NCO advice, CISA no-exploitation statement, 13:34 UTC advisory time (all marked unverified where no primary page was fetched).

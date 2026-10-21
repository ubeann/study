# study

My second brain for things I learn while building with AI tooling: version control, AI dev workflows, databases, and the tools around them.

Every topic is one Markdown note, verified against primary sources, usually with a diagram page next to it. Notes are written with Claude in [T3 Code](https://t3.gg) and checked by a script so a note without sources cannot land.

## Layout

```
notes/<area>/<slug>.md    one note per topic, frontmatter + sections (see templates/)
notes/INDEX.md            generated table of every note
diagrams/<slug>.html      diagram page for a note, built from diagrams/_template.html
templates/                concept, comparison, article, lookup
scripts/check_notes.py    validates note frontmatter, rebuilds the index
assets/                   logo and icons
.claude/skills/learn/     the /learn skill that writes notes
AGENTS.md                 pointers for any coding agent working here
```

Areas today: `ai-workflow`, `vcs`, `databases`, `tooling`. Add a directory when a note fits none of them.

## Workflow

1. Open the **Study** project in T3 Code. Run threads in the root checkout, not a worktree, so agent memory and notes stay in one place.
2. Ask the question the way you would ask a colleague: "I'm learning about merge queues, why would I use one?" or `/learn <question>`.
3. The agent greps `notes/` first, fetches the sources, writes `notes/<area>/<slug>.md` and `diagrams/<slug>.html`, runs the checker, and commits.
4. Read the chat reply for the short version. Read the note when you want the sources.

## Note anatomy

```yaml
---
title: Jujutsu vs Git
kind: comparison          # concept | comparison | article | lookup
area: vcs                 # must match the directory
date: 2026-10-21
status: draft             # draft | verified
diagram: diagrams/jujutsu-vs-git.html   # optional
sources:
  - https://github.com/jj-vcs/jj
unverified:               # claims with no fetched source; empty on a verified note
  - "LFS support status"
tags: [jujutsu, git]
---
```

`sources` must be non-empty URLs. A `verified` note cannot carry `unverified` claims. CI runs `python3 scripts/check_notes.py` on every push.

## Commands

```sh
python3 scripts/check_notes.py                # validate all notes
python3 scripts/check_notes.py --write-index  # validate and rebuild notes/INDEX.md
```

---
name: learn
description: Answer a learning question from verified sources and file it as a note in this repo. Use when the user is learning about a concept or tool, asks to explain or compare something, asks to check an article URL, or asks a one-fact question such as a price, a limit, or a release status.
---

Every answer leaves a **note** in `notes/<area>/<slug>.md`. Concepts, comparisons and articles also get a **diagram** page in `diagrams/<slug>.html`. The note is the deliverable; the chat reply summarises it.

## Steps

1. **Recall.** Run `grep -ril "<topic words>" notes/` and read every hit. An existing note is the starting point: extend it in place. Done when every note on the topic has been read.

2. **Classify** the question into one branch and copy that template from `templates/`:
   - **lookup**: one fact (a price, a cap, a release status, "is X built on Y"). No diagram.
   - **article**: the user gave a URL to explain.
   - **concept** or **comparison**: "explain X", "X vs Y", "why X instead of the usual way".

3. **Verify.** Fetch every URL the user gave, including the article or repo they linked. Each number, price, date, version, or release status in the note cites a page fetched this session, or goes under `unverified:`. Library and vendor facts go through context7 or the vendor's docs MCP before a web search. Search snippets are leads, not sources: open the page. Done when `sources:` lists every page cited and no claim is left uncited.

4. **Diagram** (concept, comparison, article). Copy `diagrams/_template.html` to `diagrams/<slug>.html` and fill the five sections. Then `html_preview` once and `html_render` once with the file's content, using the preview's contentHeight. The template already uses the T3 theme variables, so it needs no palette work, no screenshots, and no extra preview rounds unless the preview reports an error. The `dataviz` skill is for plotting real numbers, which concept diagrams rarely have.

5. **Write the note** at `notes/<area>/<slug>.md` from the template. Use an existing area directory; create one only when none fits. Run `python3 scripts/check_notes.py --write-index` and fix what it reports. Done when it exits 0.

6. **Commit** the note, the diagram, and `notes/INDEX.md` on the current branch as `docs(<area>): <slug>`. Push only when asked.

7. **Reply** in plain English, sized to the question: a one-line question gets a one-line answer plus its source; an "explain" question gets a short paragraph, what the diagram does not cover, and the note path.

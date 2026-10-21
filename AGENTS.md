# study

Personal learning notes. One note per topic in `notes/<area>/<slug>.md`, verified against primary sources, with an optional diagram in `diagrams/<slug>.html`.

- A learning question, "explain X", "X vs Y", "check this article", or a pricing or release-status question: use the `learn` skill in `.claude/skills/learn/SKILL.md`.
- Note templates live in `templates/`; the diagram page template is `diagrams/_template.html`.
- `python3 scripts/check_notes.py --write-index` validates every note and rebuilds `notes/INDEX.md`. CI runs the check on every push.

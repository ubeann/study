---
title: Jujutsu vs Git
kind: comparison
area: vcs
date: 2026-10-21
status: verified
diagram: diagrams/jujutsu-vs-git.html
sources:
  - https://github.com/jj-vcs/jj
  - https://docs.jj-vcs.dev/latest/working-copy/
  - https://docs.jj-vcs.dev/latest/conflicts/
  - https://docs.jj-vcs.dev/latest/operation-log/
  - https://docs.jj-vcs.dev/latest/glossary/
  - https://docs.jj-vcs.dev/latest/git-compatibility/
  - https://docs.jj-vcs.dev/latest/tutorial/
  - https://docs.jj-vcs.dev/latest/bookmarks/
  - https://docs.jj-vcs.dev/latest/cli-reference/
unverified:
tags: [jujutsu, jj, git, version-control]
---

## Verdict

Jujutsu (`jj`) is not a replacement for Git's storage; it is a different front end on a Git repo. Pick it when you rewrite history often (stacked PRs, fixing an older commit in a stack, splitting commits) because it removes the staging area, the stash, and the stop-and-`--continue` rebase. Stay on plain Git when you depend on hooks, submodules, LFS, or `.gitattributes`, which jj does not support, or when your team cannot absorb an experimental tool whose on-disk format may still change before 1.0.

## Side by side

| | Git | Jujutsu |
|---|---|---|
| Model | Working dir, index, commits, plus stash. Commit hash is the only identity. | The working copy *is* a commit (`@`); most `jj` commands snapshot it automatically. Every commit has a stable **change ID** plus a Git commit ID. |
| Workflow | `add`, `commit`, `stash`, `rebase -i`; a rebase that hits a conflict stops until you resolve or abort. | `jj new`, `jj describe`, `jj squash`, `jj split`, `jj edit`; descendants of a rewritten commit are rebased automatically; conflicts are recorded in the commit and the rebase "will succeed". |
| Tooling and ecosystem | Everything. | Git is the storage layer; a colocated workspace shares one working copy with `.git/` and you can mix read-only `git` commands. Branches are "bookmarks" that do not move when you create a new commit. Hooks, submodules, LFS, `.gitattributes`, partial clones: not supported. |
| Cost or limits | Stable. | README: "an experimental version control system" with "backward-incompatible changes to the on-disk formats before version 1.0.0"; started as a hobby project in late 2019 by Martin von Zweigbergk; Google is a main contributor but it is "not a supported Google product"; 32.2k GitHub stars at fetch time. |

## Where Git wins

- Feature coverage: jj lists hooks, submodules, Git LFS, `.gitattributes`, partial clones, and annotated tags as unsupported or partial.
- Stability: no format changes pending; every tool and host understands it.
- Interleaving `git` and `jj` in one repo has documented drawbacks: divergent change IDs, slow commands when there are many refs, conflicted files that Git tools cannot read.

## Where Jujutsu wins

- One place for code instead of four: the working-copy commit is snapshotted by almost every `jj` command, new files are tracked implicitly, `.gitignore` is honoured (there is no `.jjignore` yet). Switching tasks is `jj new`; nothing to stash.
- Stable identity: "change IDs ... stay the same when the commit is rewritten" (tutorial), so a stack of PRs keeps its names through amends and rebases.
- Automatic rebase: "When you modify a commit, every descendent is automatically rebased on top of the freshly-modified one" (README). `jj edit A`, fix, done; B and C follow.
- First-class conflicts: a conflicted state is stored in the commit as a logical representation, not markers; rebases and merges finish; resolve later with `jj new <commit>` + `jj squash` or `jj edit <commit>`.
- Operation log: every repo-modifying operation stores a snapshot ("view"); `jj undo` reverses the last one, `jj op restore` returns the whole repo to an earlier point, `jj op log` lists them. It also makes concurrent `jj` commands safe.

## When to pick which

| Pick jj | Pick Git |
|---|---|
| Stacked or dependent PRs you re-edit often | Repo relies on hooks, submodules, LFS, or `.gitattributes` |
| You want to try a command without fear (`jj undo`) | Team or CI tooling must stay plain Git end to end |
| Solo or small-team work in a Git-hosted repo (`jj git clone` / `jj git init` colocates by default; teammates still see normal Git) | You cannot accept pre-1.0 on-disk format changes |

Command cheat sheet (from the CLI reference): `jj git clone`, `jj git init` (colocated by default; `--no-colocate` to hide `.git` inside `.jj`), `jj log`, `jj new -m`, `jj commit -m`, `jj squash` (working copy into its parent, "very similar to `git commit --amend`"), `jj split`, `jj bookmark create x -r @`, `jj git push --bookmark x` or `jj git push -c @` (creates and tracks a bookmark), `jj undo`, `jj op log`.

## Diagram

[diagrams/jujutsu-vs-git.html](../../diagrams/jujutsu-vs-git.html): the four model differences (working copy as commit, change ID, auto-rebase of descendants, stored conflicts), the operation log, and a Git-to-jj command table. It does not cover bookmarks and remotes in depth, or the documented pitfalls of mixing `git` and `jj` writes in one repo.

## Sources

- https://github.com/jj-vcs/jj: backs the feature list (working copy as commit, operation log, automatic rebase of descendants, first-class conflicts, Git as storage layer), the experimental status and pre-1.0 format warning, origins (2019 hobby project, Google contributor but not a Google product), Mercurial/Sapling/Piper inspiration, and the 32.2k star count.
- https://docs.jj-vcs.dev/latest/working-copy/: backs automatic snapshotting, implicit tracking, `.gitignore` handling and no `.jjignore`, and the `jj new` + `jj squash` / `jj edit` conflict-resolution flows.
- https://docs.jj-vcs.dev/latest/conflicts/: backs conflicts being recorded in commits as a logical representation, the rebase succeeding on conflict, and postponed resolution.
- https://docs.jj-vcs.dev/latest/operation-log/: backs operations storing views, `jj undo`, `jj op restore`, `jj op revert`, `--at-op`, and concurrency safety.
- https://docs.jj-vcs.dev/latest/glossary/: backs change vs commit ID, bookmarks not moving on new commits, divergent changes, colocated workspaces.
- https://docs.jj-vcs.dev/latest/git-compatibility/: backs colocation being the default, mixing `git` and `jj`, and the supported/unsupported feature table (hooks, submodules, LFS, `.gitattributes`, partial clones, tags).
- https://docs.jj-vcs.dev/latest/tutorial/: backs `@` as the working-copy commit, change IDs staying stable when rewritten, `jj squash` behaving like `git commit --amend`, `jj undo`, and `jj git init` creating a colocated workspace.
- https://docs.jj-vcs.dev/latest/bookmarks/: backs `jj git push --bookmark foo` mapping to branch `foo`, and remote-tracking bookmarks.
- https://docs.jj-vcs.dev/latest/cli-reference/: backs the command descriptions and flags in the cheat sheet (`--colocate` default, `--no-colocate`, `jj git push -c`, `jj split`, `jj op log`).

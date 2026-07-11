# spec-to-ship

A Claude Code skill that manages a docs-first project end-to-end: **documentation → planning → implementation → build → test → handover** — one milestone, phase, or sprint at a time, always stopping at a review gate.

It exists for one reason: agents that "just start coding" produce code nobody specified, verified, or can resume. spec-to-ship enforces the discipline that makes agent work reviewable — spec before code, gates before claims, a memory log so any fresh session (or a different agent entirely) can pick up where the last one stopped.

## Install

```bash
npx skills add <your-github-owner>/spec-to-ship
```

Or manually: copy this folder to `~/.claude/skills/spec-to-ship/`.

## Use

```
/spec-to-ship M5            # execute milestone M5 from the roadmap
/spec-to-ship "Phase 4"     # phase/sprint/step naming all works
/spec-to-ship init          # repo has no docs? bootstrap them first
```

Or just say it: *"build milestone 2"*, *"complete sprint 12"*, *"set up docs and phases for this repo"*.

## What a run looks like

**Execute mode** (repo has a roadmap):

1. Reads the briefing, memory log, roadmap entry, and phase spec — adopting *your* doc conventions (ADRs, `ROADMAP.md`, iteration naming), never imposing its own.
2. Verifies perishable facts (model IDs, package viability) before building on them.
3. Plans task-by-task, checks for existing skills before hand-rolling capabilities.
4. Implements on a branch with a commit + checkpoint after every task — a dead session costs at most one task.
5. Runs every quality gate the project defines, boots the real app, and exercises the feature (tests passing ≠ verified).
6. Updates the memory log — including an explicit **"Not verified"** list — and stops at a review-ready branch. It never merges, deploys, or rolls into the next milestone on its own.

**Bootstrap mode** (repo has no docs): maps the repo, interviews you with the questions the code can't answer, and writes a minimal working doc set (briefing, roadmap, phase specs, technology decisions, seeded memory log) — then stops for your approval before any feature code.

## Companion skills (optional)

The skill orchestrates; these sharpen specific steps and are detected at runtime — everything degrades gracefully if absent:

| Skill | Adds |
|---|---|
| [graphify](https://github.com/Graphify-Labs/graphify) | semantic knowledge graph of the repo for cheap orientation |
| [prompt-master](https://github.com/nidhinjs/prompt-master) | optimized doc-generation brief in bootstrap |
| [ponytail](https://github.com/DietrichGebert/ponytail) | minimal-implementation discipline + over-engineering review |
| find-skills ([skills.sh](https://skills.sh/)) | buy-before-build skill search during planning |

## License

MIT — see [LICENSE](LICENSE).

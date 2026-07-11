# Changelog

## 1.0.0 — 2026-07-11

First public release.

- Execute mode: run one milestone/phase/sprint from kickoff to a review-ready branch.
- Bootstrap mode (`/spec-to-ship init`): build the doc set first on undocumented repos.
- Docs located by role, not filename — adopts existing conventions (ADRs, `ROADMAP.md` iterations, custom memory logs).
- Resilience: per-task checkpoints, optional usage-window awareness, PARKED path for honest partial handovers.
- Companion-skill integration (graphify, ponytail, prompt-master, find-skills) — all optional, graceful degradation.
- Cold-run tested on an undocumented repo and a foreign-conventions repo via skill-blind agents.

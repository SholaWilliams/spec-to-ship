# Changelog

## 1.2.0 — 2026-07-12

- Robustness & gap fixes across the entire flow:
  - **Bootstrap headless fallback:** if no user is available for the interview, produce a minimal doc skeleton with `<!-- TODO(interview) -->` markers and park the milestone — no more dead‑end blocking.
  - **Preflight brittleness:** if the companion‑skill check cannot determine installation status (no programmatic list, permission error), skip the check silently and continue. Install failures are caught immediately, falling back to doing the step yourself without retry.
  - **Ponytail‑review re‑gate:** after applying deletions, re‑run unit tests and boot check for the affected area; revert deletions if they cause a regression.
  - **Token discipline realism:** softened the absolute “must NOT burn a whole context window” to a practical guideline — note unexpectedly large docs to the user and accept the cost, or ask for a split.
  - **Memory‑log template:** added a structured template in step 8 to ensure every handoff entry captures “what changed,” “decisions with why,” “gotchas,” “not verified,” and “notes for next milestone.”
  - **GUI boot handling:** if the app requires a display or GUI automation not available, do not hang; record the feature as “not verified – requires manual GUI check” and move on.
  - **Graphify hook failure resilience:** if the auto‑rebuild hook installs but silently fails, log a warning in the checkpoint block and attempt a manual update at milestone end; note any staleness in the memory log.
  - **Scaling‑to‑a‑team consistency:** Bootstrap step 4 now includes updating `README.md` and `CONTRIBUTING.md` with a minimal setup section, closing the loop with the team scaling section.
  - **Checkpoint block now includes commit hash** (in addition to branch and milestone id) for unambiguous recovery.
- **Model selection section refined:**
  - Early‑exit if the session model is already the cheapest tier.
  - Subagent return‑shape guidance made flexible (not a rigid “file:line” rule).
  - Clarified when `find-skills` should be spawned vs. done inline.
  - Added guardrail: Haiku only for independently re‑verified output.
  - Emphasized that route‑down‑never‑up protects user spend.
- **Rework path clarified:** after review rework, run unit/integration tests for changed files; run the full suite only if core contracts changed.

## 1.1.0 — 2026-07-11

- Model auto-selection: new "Model selection — right model for the right task" section with a task→model routing table (haiku for mechanical fan-out/triage via subagents, sonnet for fact checks and implementation, strongest model for planning/docs/decisions), spawn-economics rule, and guardrails (haiku only for re-verified output, route down never up, conclusions-only subagent prompts, aliases over dated model IDs). Replaces the old one-line "Model mix" bullet.

## 1.0.0 — 2026-07-11

First public release.

- Execute mode: run one milestone/phase/sprint from kickoff to a review-ready branch.
- Bootstrap mode (`/spec-to-ship init`): build the doc set first on undocumented repos.
- Docs located by role, not filename — adopts existing conventions (ADRs, `ROADMAP.md` iterations, custom memory logs).
- Resilience: per-task checkpoints, optional usage-window awareness, PARKED path for honest partial handovers.
- Companion-skill integration (graphify, ponytail, prompt-master, find-skills) — all optional, graceful degradation.
- Cold-run tested on an undocumented repo and a foreign-conventions repo via skill-blind agents.

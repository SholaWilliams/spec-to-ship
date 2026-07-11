---
name: spec-to-ship
description: "Manage a docs-first project end-to-end — documentation → planning → implementation → build → test → handover. Execute one milestone/phase/sprint: read the phase doc, plan, implement task-by-task, run every quality gate, boot the real app to verify, update the memory log, and stop at the review gate. If the repo has no docs, bootstrap them first (using graphify-out/ and an interview if available) before any code. Trigger: /spec-to-ship <id> (e.g. /spec-to-ship M5, /spec-to-ship 'Phase 4', /spec-to-ship 'Sprint 7'), /spec-to-ship init (docs bootstrap), the legacy alias /milestone, or when the user says 'work on step N', 'implement phase X', 'build milestone Y', 'complete the sprint', 'set up docs/phases for this repo'. Do NOT trigger for one-off bug fixes or edits that aren't part of a planned phase."
---

# /spec-to-ship — Docs-first project executor

Manage a project the way this user works, from documentation to handover: spec before code, gates before claims, a review gate before the next phase. Two modes:

- **Execute mode** (default): the repo has docs and a roadmap — run one milestone from kickoff to review-ready.
- **Bootstrap mode** (`/spec-to-ship init`, or auto-detected): the repo lacks the needed docs — build them first. Docs are Milestone 0; no feature code ships until they exist.

## Companion skills (use if installed, skip gracefully if not)

This skill orchestrates; these sharpen specific steps. Before each use, check the skill exists in the available-skills list — never guess.

**How companions are referenced:** always by **name against the available-skills list** at runtime — never by install path, and plugin-namespaced variants count as present (`ponytail:ponytail` satisfies `ponytail`). The Source column is only for installing what's missing.

**Preflight install (first action of either mode):** compare this table against the available-skills list. If any are missing, offer to install them once, in a single batch, using each row's Source — then continue. If the user declines or install isn't possible, do the step yourself and move on. Never vendor copies of these skills into this one: they update independently, and a bundled copy is a stale fork by next month.

| Skill | Source | Where in the flow | What it adds |
|---|---|---|---|
| `find-skills` | `npx skills add find-skills` ([skills.sh](https://skills.sh/)) | Bootstrap step 1; Execute step 3 (Plan) | Search the skills ecosystem before hand-rolling a capability a battle-tested skill already covers |
| `graphify` | [Graphify-Labs/graphify](https://github.com/Graphify-Labs/graphify) | Bootstrap step 1; Resilience §3 | Semantic knowledge graph of the repo/docs — cheap orientation queries instead of file reads |
| `prompt-master` | `npx skills add https://github.com/nidhinjs/prompt-master --skill prompt-master` | Bootstrap step 3 | Turn interview answers into an optimized doc-generation brief |
| `ponytail` | plugin: `/plugin marketplace add DietrichGebert/ponytail`, then install `ponytail` | Execute step 5 (Implement) | Enforce the simplest implementation that works — YAGNI, stdlib-first, no speculative abstraction |
| `ponytail-review` | same ponytail plugin (bundled) | Execute step 6 (Gates) | Over-engineering review of the milestone diff: what to delete or simplify |
| `/security-review` | built into Claude Code — no install | Execute step 6 (Gates), conditionally | Security review — ponytail does **not** do security; this does |
| `/code-review` | built into Claude Code — no install | Before the review gate (step 9) | Correctness-bug review of the branch diff |

## Inputs

- **Milestone id** from the argument (`M5`, `Phase 4`, `Sprint 7`, `step 3`). If missing, find the next unstarted one in the roadmap and confirm it with the user before writing code.
- **The docs — roles, not filenames.** Locate whatever fills each role, in this order: the **briefing** (`AGENTS.md` / `CLAUDE.md` / `CONTRIBUTING.md`), the **memory log** (`docs/ai/MEMORY.md`, `NOTES.md`, a changelog-style log — newest entry is current state), the **roadmap** (root `ROADMAP.md`, `docs/*roadmap*`, `DEVELOPMENT_PLAN.md`, `docs/00-*map*`), then the **phase spec** for this milestone — which may be a separate doc *or* a sufficiently detailed section inline in the roadmap (goal + tasks + definition of done is enough). ADRs count as the technology-decisions doc. Adopt the conventions you find; never restructure existing docs into this skill's default layout. If `graphify-out/` exists, `/graphify query` is allowed for orientation, but the phase spec itself must be read raw — it is the contract.
- **Missing docs?** Only a missing **roadmap or phase spec** forces Bootstrap mode — do NOT improvise a plan from the code; tell the user why and return to Execute once docs exist and are approved. A missing *briefing* alone is a gap, not a mode switch: note it, continue Execute, and offer to write one at the handover.

## Bootstrap mode — build the docs first

Goal: produce just enough documentation that a fresh session (or a cheaper model) can run Execute mode without guessing. Not a wiki — a working contract.

**Perspective rule (applies to every doc written):** capture the **what** (the feature/behavior), the **why** (the decision and its rationale), and the **how** (approach, contracts, constraints). Never line-by-line implementation detail — code documents itself; docs document intent.

1. **Understand the repo before asking anything.**
   - If `graphify-out/` exists: query the graph first (`/graphify query`, `graphify-out/wiki/index.md`) to map architecture, entry points, subsystems, and existing conventions. Verify anything load-bearing against real files.
   - If not: offer `/graphify .` for large repos; otherwise map it with Glob/Grep/targeted Reads (or `/onboard` if there's no CLAUDE.md at all).
   - Run the real build/test commands once to learn what the actual gates are. If the project defines none, run the closest smoke check (build it, invoke it once) — and defining gates becomes part of the doc set in step 4.
   - If `find-skills` is installed: note the project's domain (e.g. PDF handling, Excel, 3D, payments) and search for existing skills covering it — a found skill becomes a technology decision in step 4, not custom code in some future milestone.
2. **Interview the user — the great questions.** Ask only what the code can't answer, in one batch (AskUserQuestion or a compact list); one follow-up round is allowed where an answer was thin. E.g.:
   - What is this project *for*, and who uses it? What does "done" look like?
   - What's in scope for the next 1–3 milestones vs. explicitly out of scope?
   - Which decisions are already made and binding (stack, libraries, providers, formats)? Which libraries were *rejected*, and why?
   - What are the contracts that must not silently change (persisted files, schemas, APIs, CLI flags)?
   - What are the quality gates ("done" = which tests, lint, boot check)? Any live/paid tests to never run?
   - Any known landmines, ⚠️ perishable facts (model IDs, endpoints), hardware/keys constraints?
3. **Compose the doc-generation brief.** Distill the repo map (step 1) + interview answers (step 2) into a single brief that will drive the doc writing: project purpose, audience, scope, binding decisions, contracts, gates, landmines, and the perspective rule. If `prompt-master` is installed, invoke it to optimize this brief as a prompt; either way, save the result to `docs/ai/BOOTSTRAP_BRIEF.md` so the doc set is reproducible and auditable, then write the docs *from the brief*.
4. **Write the doc set** (small, numbered, each with a one-line purpose header). **Conventions rule:** if the repo already has doc conventions — a different docs layout, ADRs, a differently-named memory/changelog file, its own task-ID scheme — adopt those and map the roles below onto them; never restructure existing docs to match this list. The layout below is the *default for greenfield repos only*:
   - `AGENTS.md` — briefing: reading order, unbreakable rules, verification commands. (`CLAUDE.md` points to it.)
   - `docs/00-roadmap.md` — milestones M1…Mn, each with goal, task IDs (T-numbers), definition of done, and **dependencies on other milestones** (so independent ones can run in parallel later). **Size rule:** a milestone must fit one session — one subsystem, roughly 3–6 tasks; if it won't fit, split it in the roadmap now.
   - `docs/NN-<area>.md` phase/spec docs — what/why/how per feature area; binding decisions get D/TD numbers; verify-at-implementation facts get ⚠️.
   - `docs/04-technology-decisions.md` (or equivalent) — accepted and **rejected** dependencies with reasons (skills found in step 1 count as accepted technology).
   - `docs/ai/MEMORY.md` — memory log seeded with a "Current state" entry.
   - For an existing codebase, docs describe reality first (as-built), then the plan; flag any place code contradicts stated intent.
5. **Review gate.** Present the doc set; the user approves or edits. Do not start M1 until approved. Docs are commit-worthy (`docs/<area>-<short>` branch, Conventional Commits) when the user says so. Once approved and committed, if graphify is installed, run `/graphify .` on the repo so the doc set is semantically searchable before M1 starts (Resilience §3).

## Token discipline (this skill must be cheap to run)

A milestone must NOT burn a whole context window on reading. Rules:

- **Read sections, not documents.** Use the doc map / table of contents to pull only the sections the phase doc cites. Never read a full spec "for background".
- **Memory log: newest entries only.** Read the "Current state" section and the decision entries touching this milestone's subsystem; skip the rest.
- **Targeted tests while iterating** — run only the area you're touching, with the project's own runner (`pytest tests/unit/<area> -x`, `npm test -- <pattern>`, `cargo test <module>`, `go test ./<pkg>`); the **full gate suite exactly once** at the end. Never paste full passing-test output — report counts; paste only failures.
- **Don't re-read files already in context.** Quote only the lines that matter.
- **One milestone per session** (Bootstrap counts as a milestone). The memory-log entry (step 8) is the handoff — it must let a fresh session (or `/clear`) resume without re-reading history.
- **Companion skills are surgical, not ambient.** Invoke each at its designated step only; don't run ponytail-review or code-review more than once per milestone.
- **Model mix:** bootstrap + planning (steps 1–3) benefit from the strongest model; implementation (steps 4–7) runs fine on Sonnet because the plan + docs + gates carry the quality. Suggest the switch at the plan/build boundary if the user controls the model.

## Resilience — checkpoints, limit awareness, live graph

Sessions can die without warning (the 5-hour usage window, a crash, a closed laptop). The rule: **the on-disk state must always be resumable by a fresh session — or a different agent/provider entirely.** All artifacts (docs, memory log, git) are plain markdown + commits; nothing here is Claude-specific.

1. **Checkpoint after every task, not just at milestone end.** After each task's commit, update a single `## Checkpoint` block at the top of the memory log: milestone id, branch, tasks done, task in progress, exact next action, anything not yet verified. Overwrite it each time (it's a scratch slot, not history); the full step-8 entry replaces it at milestone end. A hard cutoff then costs at most one task.
2. **Usage-window check** (only applies on plans with a rolling usage window, e.g. Claude Code Pro/Max 5-hour blocks — skip silently on API/pay-per-token setups). The model cannot see the limit directly (`/usage` is a terminal dialog). Instead, at milestone kickoff and again every 2–3 tasks, run `npx -y ccusage@latest blocks --active --json` — it reads the local Claude Code transcripts (no key, no network beyond npx) and reports the active block's `endTime`. If less than ~30 minutes remain, stop implementing and run the PARKED path *now*, while there's budget to write it. Treat the numbers as a heuristic (ccusage infers blocks from timestamps); if the command is unavailable or errors, fall back to asking the user at kickoff how much window remains; if the user isn't available either, skip the check entirely — the checkpoint discipline in §1 is what actually makes a cutoff cheap, and it applies on every setup, window or not. Honor "park it" at any moment.
3. **Auto-graphify rides the checkpoints.** If the `graphify` skill is installed: when `graphify-out/` doesn't exist yet, build it once at the earliest useful moment — right after Bootstrap doc approval (graph the doc set) or at Execute step 1 on an existing repo — so semantic search exists from day one. Then keep it live automatically: at step 4 (branch), install graphify's post-commit auto-rebuild hook (graphify `references/hooks.md`) so every checkpoint commit refreshes the graph; if the hook can't be installed, run `/graphify <root> --update` after every 2–3 task commits instead. If graphify isn't installed, skip silently — it's an accelerator, not a gate.

## Steps (Execute mode)

1. **Orient.** Run the companion-skill preflight and the kickoff usage-window check (Resilience §2); if a `## Checkpoint` block sits atop the memory log, resume from it instead of re-planning. Then read briefing → memory log (per token discipline) → roadmap entry for this milestone → phase doc sections it cites. Extract: task list (T-numbers / action items), definition of done, binding decisions (D-numbers, TD-numbers), contracts that must not change, and any ⚠️ verify-at-implementation items. If the roadmap or this milestone's phase spec is missing → Bootstrap mode (per Inputs; a missing briefing alone is not). **Drift check:** if the phase doc contradicts the actual code, stop and surface it — the doc gets corrected (with the user) before code follows it. **Size check:** if the milestone clearly won't fit one session, propose a split in the roadmap before starting.
2. **Verify ⚠️ items first.** Model IDs, package viability, external endpoints — check them for real (web search / trial install) before building on them; update the doc with findings.
3. **Plan.** Produce a task-ordered plan mapping each task to files. **Buy-before-build:** if a task is a common capability (document generation, spreadsheet work, scraping, charts…), and `find-skills` is installed, search for an existing skill first — installing a vetted skill beats hand-rolling, but it goes through the technology-decisions doc like any dependency. Anything touching a persisted format or documented contract = versioning + migration, never edit-in-place. A task with no doc section = write the doc section first, same branch. Get plan approval if the harness supports plan mode or the project requires review gates.
4. **Branch.** `<type>/<milestone>-<short>` (e.g. `feat/m5-memory`), Conventional Commits, one coherent commit per task or task-group. If graphify is installed, set up the post-commit auto-rebuild hook here (Resilience §3).
5. **Implement task-by-task.** After each task: commit, update the `## Checkpoint` block, and every 2–3 tasks re-run the usage check (Resilience §1–2). If `ponytail` is installed, activate it for the implementation work — simplest solution that passes the gates, stdlib before dependencies, no speculative abstraction; the phase doc defines *what*, ponytail keeps the *how* minimal. Tests alongside code. New dependency? Check the project's technology-decisions doc first — rejected libraries stay rejected; otherwise ask.
6. **Run every quality gate** the project defines (look in AGENTS.md/CLAUDE.md/CI workflow). If it defines none, run the closest smoke check (build + boot) and record "no gates" as a doc gap in the memory log. All green or say exactly what's red. Never run live/network-key test markers unless the user asks. Then two diff reviews on the milestone's changes: **`ponytail-review`** (over-engineering — apply deletions that don't change behavior), and **`/security-review`** *if* the milestone touched input parsing, auth, secrets, file/network I/O, or anything user-facing (ponytail is minimalism, not security — don't conflate them).
7. **Boot the real thing.** Run the actual app (offscreen/screenshot for GUI, real invocation for CLI/API) and exercise the milestone's feature. Tests passing is not verification.
8. **Record.** Append to the memory log: what changed, decisions made **with why**, gotchas, and — critically — an explicit **"Not verified"** list (no hardware, no keys, never heard/seen it live). If implementation diverged from the phase doc, update the doc too (what/why level) — docs and code must not drift apart. Prune superseded entries, and delete the `## Checkpoint` block — this full entry replaces it.
9. **Stop at the review gate — the handover.** Optionally run `/code-review` on the branch diff first and fix confirmed findings. Present: tasks done, gates status, review findings handled, what was verified vs. not, the branch name, and anything the dev must do that the agent can't (deploy, rotate keys, hardware tests). Do **not** open the PR, merge, deploy, or start the next milestone unless asked. Deployment is the dev's; the skill's job ends at a review-ready branch.

## Failure and rework paths

- **Parked milestone.** If a gate is red and unfixable in-session, discovery invalidates the plan, or context is running out: stop implementing, commit what's coherent, and write a memory-log entry marked **PARKED** — done tasks, remaining tasks, the blocker, and the exact next action. Present it honestly; never pad a partial milestone to look review-ready.
- **Rework after review.** Review feedback returns to step 5 on the **same branch** (`fix:`/`refactor:` commits), then re-run the gates touched (full suite again only if contracts changed) and append a short rework entry to the memory log. Feedback that changes *scope* is not rework — it's a roadmap edit, agreed with the user first.
- **No user available** (headless, CI, subagent run): anything that is merely an *offer* — companion installs, `/graphify .`, model-switch suggestions — is skipped silently and the run continues. Anything that requires a *decision* — interview answers, plan approval, drift resolution, scope or dependency sign-off — is never invented: record the questions and take the PARKED path.
- **Recurring feedback is a process bug.** If the same correction shows up in two milestones, encode it — into a gate, a doc rule, or this skill — instead of repeating it in chat.

## Scaling to a team

The same doc set onboards a human: AGENTS.md + roadmap + memory log is the reading order. Additionally, once more than one person (or agent) works the repo:

- Keep a short **setup section** (clone → install → run → gates) in the README or CONTRIBUTING.md — AGENTS.md is written for agents, humans need the two-minute version.
- **Parallel work follows the roadmap's dependency lines**: two milestones may run concurrently only if neither touches the other's contracts; memory-log entries are keyed by milestone id to merge cleanly.
- **Move the gates into CI.** A gate report from an agent (or a teammate) is a claim; a CI run is a fact. The review gate should show CI green, not just a local transcript.

## Output

- **Bootstrap mode:** an approved doc set (briefing, roadmap, phase docs, tech decisions, seeded memory log) plus the saved bootstrap brief — enough that Execute mode runs without guessing.
- **Execute mode:** a review-ready branch — all milestone tasks implemented (minimally, if ponytail is present), all gates green (or honestly reported), reviews run, app booted and exercised, memory log updated, and a short handover report in the exact shape of step 9.

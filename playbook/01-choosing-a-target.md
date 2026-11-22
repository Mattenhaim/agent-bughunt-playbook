# Step 1 — Choosing a target

**Goal:** one module, one hard boundary, one session. Vagueness is the #1 reason agent hunts produce nothing.

## Pick by bug-habitat, not by fame

Good hunting ground has any of these traits:

- **Serialization boundaries** — import/export, JSON/JSONL parsing, file names vs resource ids. Data crosses a trust/format boundary; one malformed file or one swapped identifier breaks invariants.
- **Lifecycle handoffs** — create → update → delete → move. Check-then-act gaps, stale caches, ownership transfer edge cases.
- **Config-driven behavior** — CORS lists, feature flags, path templates. The code trusts configuration that users type with extra spaces.
- **Recently changed code** — `git log --since=<date> -- <paths>`; fresh code has fewer eyeballs.
- **Module-level consistency checks** — places where two functions compute the same precedence rule differently (a duplicate state both implementations disagree on is itself evidence the state is unintended).

## Draw the boundary before starting

Write the scope down and hand it to the agent verbatim:

- module/directory allowlist (e.g. `packages/cli/src/webhooks/**`) and hard exclusions (e.g. skip `source-control.ee` this round if it was covered last time);
- what "confirmed" means: *a test that fails on upstream `master`* — everything else is "suspected" and must be labeled as such;
- a same-failure loop limit: if the agent fails the same verification twice, it records the blocker and moves on. Endless retry loops burn the budget and produce rationalized "findings".

## Budget the session

A productive read-only audit session covers one module deeply (about 2–5k lines) rather than five shallowly. Two sessions on adjacent modules beat one sprawling one — in the n8n case, session 1 (source-control.ee) produced 5 PRs and session 2 (variables/webhooks/CORS) produced 3, and the split made each report self-contained.

## Reuse the clone, don't fork the workflow

If a previous session left a working clone with dependencies installed, re-use it: fetch upstream `master`, create a fresh branch, and keep prior branches untouched. Dependencies are expensive; branches are cheap.

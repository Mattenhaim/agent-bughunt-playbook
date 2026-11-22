# Step 8 — When the agent must ask a human

**Goal:** outward-facing and hard-to-reverse actions are human decisions. Everything else is the agent's job.

## Always human-approved (non-negotiable)

- **First push of any branch**, and every force push (`--force-with-lease` included — it rewrites remote state);
- **opening issues and PRs** on the target project (after showing the draft);
- **any comment or reply** — on issues, PRs, review threads, bot threads;
- **closing/reopening** PRs or issues, editing titles of things that exist;
- deleting branches, rewriting history, anything touching other people's PRs.

The n8n hunt enforced this with one rule: *the agent prepares everything and stops; the human says "go" once per action class.* Each approval was scoped ("only these three new branches", "one-time: create these three issues") — scoped approvals avoid approval-fatigue mistakes.

## What the human should see before saying yes

- the exact commands that will run;
- the draft PR/issue text;
- the dedup search results;
- the self-check output (no secrets, no local paths, no unproven claims — see the [security checklist](09-security-checklist.md)).

## What the agent decides alone

- reading, searching, reproducing;
- local branches, commits, stashes;
- running test suites, typecheck, formatters;
- disproving its own suspicions.

## The one-time-approval pattern

Batching several scoped actions into a single human decision ("create these three issues, edit these three PR bodies to link them") works well *after* trust is established by the first rounds. It does not replace the rule — each batch still needs the drafts shown first.

## If blocked, report, don't improvise

When a same-failure loop repeats twice, or a step needs an approval you don't have: stop that thread, record the blocker, and move to the next item. The n8n hunt hit this with a missing `gh` CLI (installed portable, no system changes) and a missing round-1 materials directory (re-derived from GitHub instead) — both were worked around *within* the approved scope, and both were reported.

# Step 5 — The minimal fix

> **In one line:** one bug per branch, fix the seam instead of the symptom, reuse the project's own vocabulary and validators, and verify the sibling surface (full test dir, typecheck, formatter, plus a positive control that the fix doesn't over-block).

**Goal:** the smallest change that makes the failing test pass without narrowing the contract or breaking siblings.

## One bug, one branch

Branch off the base (`hark/<topic>-<shortname>` or your own convention), one commit per logical fix. Never stack two bugs on one branch — the second bug's CI noise blocks the first one's review.

## Fix the seam, not the symptom

Examples from the n8n hunt:

- The variable-move duplicate was a missing validation, not a dedup pass at read time: add the check at the mutation point, reusing the project's existing validator (`validateUniqueVariable`), validating the key that *will land* in the destination (`variable.key ?? existingVariable.key`) so move+rename is not over-blocked.
- The dynamic-webhook CORS bug was a node-matching failure, not a CORS rewrite: add a match arm keyed on the webhook row's `webhookId` (only dynamic rows carry it, so it cannot collide with static rows) instead of rewriting the whole lookup.
- The CORS trim bug was one split without trim: `.split(',').map(o => o.trim()).filter(o => o.length > 0)` — plus the comment explaining *why* (users type spaces).

## Reuse the project's own vocabulary

Error messages from the same validator, scopes from the same map, helpers the neighbors already use (`getRegistrationsFromPath` instead of hand-rolling a second resolver). A fix that reads like it was written by the module's author is the goal.

## Verify the sibling surface

Before declaring done:

- the full test directory that touches the module (not just the new test file);
- the package `typecheck`;
- the project's formatter/linter — and beware environment artifacts (CRLF checkouts on Windows, stale build output) that fake both failures and passes. Verify committed content in an LF clone when in doubt.
- the "does the fix over-block?" edge: write the positive-control test before committing.

## Commit message

Match the project's convention (n8n: `fix(core): Imperative summary, no period`). The commit is the changelog entry — write it for the release notes reader.

---

**Next:** [Step 6 — Writing the PR (and its issue)](06-pr-description.md)

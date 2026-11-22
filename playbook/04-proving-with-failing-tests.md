# Step 4 — Proving the bug with a failing test

**Goal:** a test that fails on upstream `master`. This is what separates a finding from an opinion, and it is the single strongest signal a maintainer can get.

## Match the project's test idiom

Read the neighboring tests first and copy their shape: the same mock builders, the same env-var setup, the same assertion style. In n8n that meant reusing the existing `mock<VariablesRepository>()` scaffolding and the `vitest-mock-extended` conventions instead of inventing a new harness. A regression test that looks native gets merged; one that looks grafted gets re-litigated.

## Test the bug, not the fix

Each test should assert the *observable contract* that `master` breaks:

- the move is rejected with the project's own error message (`rejects.toThrow('...already exists...')`) and the repository write never happens;
- a dynamic-path webhook returns the configured Allowed Origins object;
- the CORS response header echoes the allowed origin, not the first list entry.

Avoid asserting implementation details (which mock got called with which internal shape) — those break on legitimate refactors and annoy reviewers.

## Cover the bug's edges while you are there

The n8n sessions added, alongside each failing case:

- a **positive control** (the same operation against a clean scope still succeeds) — proves the fix doesn't over-block;
- adjacent **edge cases the fix touches** (move + rename validates the *new* key, not the old one);
- a **regression guard** for the sibling implementation with the same defect (the same `findAccessControlOptions` bug existed in both the live-webhook and test-webhook managers — three tests, one branch).

## Name the failure mode in the test name

`rejects the move when the destination project already has the same key` reads like the bug report it is. Future readers bisect by test name.

## Keep the failing-test run reproducible

Record the exact commands in the PR body (`cd packages/cli && pnpm vitest run <paths>`), including any env vars the project's test script needs. A maintainer should be one paste away from seeing the same red you saw.

# Step 3 — Reproducing the bug

**Goal:** turn "the code looks wrong" into "here is exactly what happens".

## Reproduce at the cheapest level that proves the point

Prefer, in order:

1. **Unit level** — call the function directly with the tricky input. Fast, deterministic, and it becomes the skeleton of the regression test.
2. **Service level with mocks** — when the bug lives in orchestration (ownership checks, scope checks), mock the repositories the way the existing test suite does.
3. **Integration level** — only when the bug is in the seam itself (e.g. importer against a real SQLite database). n8n's test setup made this a one-liner: sqlite in-memory + the package's test env vars.

## Beware environment artifacts that masquerade as bugs

Real example: running `biome check` on a Windows checkout flagged *every line* of a file — that was `core.autocrlf=true` turning the working tree CRLF, not a formatting problem. The fix for the *verification*, not the code: check the committed content in an LF clone (`git clone -c core.autocrlf=false --shared`), or pipe `git show HEAD:<file>` into the tool. Another: a stale `dist/` made five unrelated type errors appear; building the workspace package first turned the "regression" into a clean baseline. Always run the same check on **clean `master`** before calling something a finding.

## Prove the negative, too

For each fix later, you want: *test fails before the fix, passes after*. The cheapest way to keep that evidence honest is the stash dance — `git stash push -- <impl files>` leaves the new tests running against the unfixed implementation, and the expected failures become your proof. Restore immediately (`git stash pop`).

## Timing

If reproduction needs a race (two concurrent writes), don't chase it in the PR. The deterministic seam — the missing validation — is the bug; say so in the report and cover it at the seam level.

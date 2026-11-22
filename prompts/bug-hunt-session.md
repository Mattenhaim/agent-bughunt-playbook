# Prompt: bug-hunt session (read-only audit)

The master prompt that starts a hunt. Fill the `<...>` placeholders, run the agent in read-only/plan mode.

```text
Task: find real logic bugs in <project>/<module-or-dirs> (read-only + local only; no pushes, no posts).

Background: <prior sessions, modules already covered, related known issues to skip>

1) Working setup: <clone path to reuse>. Fetch upstream master first; work on a local branch
   <branch-name>; do not modify other branches or the main working tree state.

2) Target scope (hard boundary): modules/dirs <allowlist>; excluded <denylist>.

3) Method: read the code for real logic errors — boundary conditions, null/undefined, ordering,
   id-vs-name confusion, check-then-act gaps, stale snapshots across loops, permission misses.
   Every candidate must state: file:line, the violated invariant, and a concrete way to make a
   test fail on upstream master. Anything unproven is "suspected" and must be labeled as such.

4) If a verification loop fails twice, record the blocker and move to the next candidate.

5) Forbidden: push, open issues/PRs, comment anywhere, use or print credentials, paid APIs,
   changing global git config.

6) Deliverable: <report-path> — per finding: module, file:line, failing-test plan, root cause,
   suggested minimal fix, dedup status. Then summarize in-session.
```

## What makes this prompt work

- **"read-only + local only"** up front stops the agent from "helpfully" fixing things mid-audit.
- **A hard scope boundary** keeps the budget on one module and prevents re-treading covered ground.
- **"failing test on upstream master" as the definition of confirmed** forces reproduction discipline.
- **The two-strikes rule** bounds failure loops — the agent records and moves on instead of burning the session on one broken verification.
- **The deliverable format** makes the write-up free: the report entries are PR/issue bodies in waiting.

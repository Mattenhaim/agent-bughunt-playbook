# Prompt: draft the bug issue and PR body from a proven fix

Use when the failing test is green and the fix is verified — before anything is pushed. Nothing is posted; the human reviews the drafts.

```text
From the hunt report entry below, draft (do NOT post):

1) A bug issue for <owner>/<repo>, following the repo's issue template exactly:
   fields <list template fields>; reproduction steps that run on any machine (no local paths);
   honest Debug Info/Environment (versions, OS, hosting); titles as plain sentences of the
   OBSERVABLE misbehavior.
   If a closed issue describes the same symptom, add one factual sentence: what it claimed,
   what you verified (tag/file), and that it still reproduces on master.
2) The PR body following the repo's PR template: Summary (what/why/key choice, imperative),
   How to test (exact commands + which tests failed on master), Related issues
   ("Fixes #<issue>" for the bug you filed; "Related #<n>" for prior art only).
3) A PR title in the repo's convention (<type>(<scope>): Imperative summary, no period).

Constraints:
- run the repo's dedup search first; if a duplicate exists, use the existing issue number;
- no exaggeration ("Open — awaiting review" is the only status language until maintainers act);
- no local paths, no credentials, no unproven findings;
- show all three drafts and stop. Do not create/edit anything.

Hunt report entry: <paste entry>
```

## Notes

- **Drafts stop at the human.** The agent has everything staged: body files, titles, the exact `gh` commands. Approval becomes one word.
- The issue is filed *before* the PR so the PR body can say `Fixes #<issue>` — the linkage is what auto-closes the bug on merge and makes the paper trail bidirectional.
- Keep the issue about the *bug* and the PR about the *fix*; the two texts overlap but serve different readers.

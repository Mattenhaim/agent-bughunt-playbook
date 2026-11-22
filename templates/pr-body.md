# Template: PR body (n8n-style, survived real use)

Adapt the section headers to the target repo's `.github/pull_request_template.md` — the structure below matches n8n's and generalizes well.

```markdown
## Summary

<One sentence: what changes and where. Imperative mood, e.g. "Reject moving a variable into a
scope that already holds the same key.">

<Paragraph: why — the broken invariant and the user-visible consequence. Contrast with previous
behavior.>

<Optional paragraph: the main design choice, if non-obvious — e.g. "validate the key that will
land in the destination, so move+rename is not over-blocked".>

## How to test

<Numbered UI/API steps if the change is user-facing, otherwise jump to commands.>

Added tests in `<test-path>`:

- <test 1 — the failing-on-master case>
- <test 2 — positive control>
- <test 3 — adjacent edge case>

The first test fails on master.

```
cd <package-dir>
<exact test command>
```

## Related Linear tickets, Github issues, and Community forum posts

Fixes #<bug-issue-number>.
Related #<prior-art-number> — <one factual clause about the relation>.
```

## Rules that earned their keep

- **Never delete existing body content** when updating a PR (bots append their own blocks; keep them).
- **"The first test fails on master"** — one sentence, strongest review signal in the whole body.
- **Exact commands**, including env vars the test script needs. One paste, same red.
- **`Fixes #` only for the bug this PR actually closes; `Related #` for everything else.**
- Titles follow the project's convention (`fix(core): Imperative summary, no period`) — the title is the changelog entry.

# Step 9 — Pre-flight security & honesty checklist

**Goal:** nothing in a commit, PR, issue, or public repo can leak secrets, private paths, or unproven claims.

## Grep before every public artifact

Run over every file that will leave the machine:

```
ghp_  github_pat_  gho_  ghs_  ghuv_     # GitHub token formats
xoxb- xoxp- AKIA                      # other common secret prefixes
@gmail.com                             # personal emails
[A-Za-z]:\\                           # absolute Windows paths
/home/ /Users/                        # unix home paths
agent_  sess_                         # internal session/agent ids
```

Expect **zero** matches. The n8n hunt repo self-check pattern: report the command and the (empty) result in the summary, so the human sees the audit, not just a claim of cleanliness.

## Content rules

- **Proven bugs only.** Anything without a submitted PR/issue stays out of public repos; "suspected" findings live in the private hunt report or nowhere.
- **State as it is.** "Open — awaiting review", never "merged", never "accepted". Live-check every number you cite against GitHub before publishing.
- **Small snippets only.** Quoting a few lines of the target project to explain a root cause is fair use; copying whole functions into your repo is not. Describe the fix; link the PR for the diff.
- **No personal emails** in docs or commits — use the account's noreply address.
- **No local paths** (`E:\...`, `/Users/...`) — write "the local clone", "a scratch directory".

## Operational hygiene

- Never print, echo, or redirect token values — including into log files that later get shared.
- Keep tools and materials in separate scratch directories; treat downloaded archives as untrusted.
- Don't read or touch directories that might hold credentials (e.g. a folder literally named for tokens) — you don't need them, and accidental exposure is forever.
- Scope every remote action: new branches only, no deletions, no force without an explicit scoped approval.

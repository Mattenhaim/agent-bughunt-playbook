# Template: bug issue (n8n-style bug-report form fields)

Follow the target repo's `.github/ISSUE_TEMPLATE/*` — below are n8n's fields with fill-in guidance that survived real filing.

```markdown
**Bug Description**

<Clear, concise description of the misbehavior — observable, not implementation-internal.
Name the invariant the code breaks. One paragraph on impact: what the user/workflow sees.>

**To Reproduce**

1. <Setup step — state preconditions explicitly (license, feature flag, existing data).>
2. <Action step — exact request/UI action, including the tricky input verbatim
   (e.g. the body JSON, the path template, the list with the space after the comma).>
3. <Observe step — what appears/wrong value/which header.>
4. <Control step — the same setup WITHOUT the tricky detail works, isolating the cause.>

**Expected behavior**

<What should happen instead, phrased as a contract. If a validation error exists elsewhere in
the product for the same condition, quote it — reuse of the project's own vocabulary is evidence
the behavior is unintended.>

**Debug Info**

```
n8nVersion: <from Help > About, or the master commit you tested>
database: sqlite
n8n_executions_mode: main
```

<One line: the behavior is server-side logic and is not specific to this environment.>

## Environment

**Operating System:** <OS> (affects all platforms; the issue is in <subsystem>)
**n8n Version:** <exact version or master commit + date>, still present on current master (<commit>)
**Node.js Version:** <version>
**Database:** SQLite (default)
**Execution mode:** main (default)
**Hosting:** self hosted
```

## Rules

- **Reproduction runs on any machine** — no local paths, no machine-specific state.
- **Include the control step**: "without the tricky detail it works" is half the proof.
- **If reviving a closed issue**, one factual sentence in the description: what was claimed, what you verified (file/tag), and that it still reproduces on master. No argument, just the audit.
- **Anonymous dedup search first**; if a duplicate exists, comment nothing — use the existing issue.
- Labels are the triagers' job; don't force-add tags.

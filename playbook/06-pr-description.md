# Step 6 — Writing the PR (and its issue)

**Goal:** make the maintainer's first read land the whole story in under a minute.

## Use the project's template, exactly

n8n's PR template is `Summary` / `How to test` / `Related Linear tickets, Github issues, and Community forum posts`. Fill it in that order, imperative mood, no hype. The bug-issue template (Description / To Reproduce / Expected behavior / Debug Info / Environment) maps almost one-to-one from your hunt report — generate it, don't retype it.

## Summary structure that works

1. One sentence: what changes and where, imperative ("Reject moving a variable into a scope that already holds the same key").
2. Why: the invariant broken and the consequence ("`$vars` resolution becomes order-dependent").
3. The main choice, if non-obvious ("validate the key that will land in the destination, so move+rename is not over-blocked").

## How to test = the maintainer's paste buffer

The exact commands, in fenced blocks, plus the one-line statement *"the first two fail on master"*. If reproduction needs UI steps, number them so they can be followed without reading the code.

## Link issues with the right keyword

`Fixes #<issue>` in the PR body auto-closes the issue on merge. Use `Related #<n>` for context-only references (prior art, sibling bugs). If a GitHub issue exists for the bug, file it *first* and link it — many repos gate or triage on issues, and the linkage makes the paper trail bidirectional.

## File issues with the same care

Anonymous dedup search first (see [the prompt](../prompts/dedup-search.md)). Then the bug template, with a reproduction that does not depend on your machine's paths, honest version info, and — when reviving a closed issue — one factual sentence about what was checked and what still reproduces (see [case 8](../case-studies/n8n/08-webhook-cors-origin-trim.md): #18528, closed as fixed in 1.109.0, parsing code unchanged on master).

## Titles follow the project's conventions

n8n PR titles are `type(scope): Summary` with the summary capitalized, imperative, no period — the title becomes the changelog entry. Issue titles are plain sentences describing the observable misbehavior, not the implementation.

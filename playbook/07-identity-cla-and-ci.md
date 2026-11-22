# Step 7 — Identity, CLA, and CI traps

**Goal:** don't lose a day to the mechanical layer. All three traps below were hit for real in the n8n case.

## Trap 1: commit authorship breaks the CLA check

EasyCLA links commits by author email to signed accounts. Our first push used a placeholder identity (`hark <hark@localhost>`) set repo-locally because global git config was off-limits — every PR went red: `CLA Check: FAILURE (1 unlinked commit(s))`, while "Verify CLA signatures" stayed green (the *account* was fine; the *commits* weren't linked).

The fix pattern, once the human approves:

1. set the identity **repo-locally** to the GitHub account's noreply email (`git config user.email "12345+<login>@users.noreply.github.com"`);
2. `git commit --amend --no-edit --reset-author` per commit;
3. prove the rewrite is content-only: `git diff <old-sha> <new-sha>` must be empty;
4. update the remote with a *leased* force: `git push --force-with-lease=<branch>:<old-sha> origin <local>:<remote>` — the lease fails if anyone else moved the branch meanwhile.

**Prevention:** set the correct repo-local identity *before* the first commit, not after the first push.

## Trap 2: line endings fake a formatting failure

On Windows with `core.autocrlf=true`, the working tree is CRLF but formatters (biome in n8n's case) expect LF, so `check` fails on every line of every file. Verify committed content instead:

- `git clone -c core.autocrlf=false --no-checkout --shared <local-clone> <scratch>` — a zero-cost shared clone;
- run the formatter there, or pipe `git show HEAD:<file>` into `--stdin-file-path`.

Same class of trap: stale `dist/` output faking five unrelated type errors — build the dependency package, then re-run typecheck on `master` to establish the true baseline.

## Trap 3: CI that never starts (and AI reviewers that do)

Fork PRs in n8n don't run the heavy workflows until a maintainer approves them — a checks list with only CLA/Aikido/cubic is *normal*, not a failure. What does run immediately:

- **EasyCLA** — author email matters (Trap 1);
- **AI code reviewers** (cubic in n8n's case) — they post inline comments within minutes. Read them like a reviewer, not like gospel: our real set was one valid catch (a `HEAD`-method lookup gap, fixed with tests) and one vague-but-cheap-to-satisfy ask (a regression test for the removed substring matching). Fixing both cost one commit; **never reply in the bot's thread** — let the commit speak. If a comment is wrong, record why in your own notes and move on.

## The sequencing that avoids re-work

1. correct repo-local identity → 2. tests + typecheck + formatter on committed content → 3. dedup search → 4. human approval → 5. push → 6. open issue + PR with links → 7. watch checks; treat AI-review comments as a triage list, not a chat.

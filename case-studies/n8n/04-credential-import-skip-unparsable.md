# Case 4 — Pull and policy preview still crash after the first malformed-file fix

**PR:** [#44750](https://github.com/n8n-io/n8n/pull/44750) · **Module:** `source-control.ee` · **Status:** Open

## Problem

After [#43952](04-source-control-skip-unparseable-files.md) fixed the *status-side* readers, status finally reported a pullable state — but the pull itself still failed at the credential import step (after workflows had already been imported), and the content-import policy preview threw a 500 on the same malformed file. The first fix made the failure surface later instead of fixing the whole class.

## Root cause

Two more bare `jsonParse` reads of `credential_stubs/*.json` remained: `importCredentialsFromWorkFolder` (pull) and `previewContentImportPolicy` (pull preview when content-import policies are configured). Same missing batch-isolation pattern as case 1, in different call sites.

## Reproduction

1. Push once so `credential_stubs/` exists.
2. Break one credential file's JSON.
3. Run a pull and the pull-status preview endpoint.
4. Before (with #43952 applied): pull fails mid-import; preview returns 500. After: both complete, good credentials import, the log names the broken file.

## Fix approach

Wrap both readers in the same warn-and-skip pattern as case 1, plus two edge cases the catch-only fix would miss:

- files that parse to a falsy value without throwing (`jsonParse('null')`);
- credential files without an `id` on import.

Tests: `skips a credential candidate with invalid JSON instead of failing the pull` and `skips a credential file with invalid JSON instead of failing the preview`.

## Lessons

- **Fix the class, not the call site.** After the first guard, ask "who else reads the same untrusted input?" — the audit found the answer by searching the module for the same parse call.
- **Partial fixes move the failure downstream**, which can look like a new bug. The follow-up PR framed itself as completing the prior one (related, not duplicate).
- `jsonParse('null')` silently returning falsy is a separate hazard from throwing — cover both.

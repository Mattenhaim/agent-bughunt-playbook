# Case 1 — One malformed file breaks source-control wholesale

**PR:** [#43952](https://github.com/n8n-io/n8n/pull/43952) · **Issue:** [#43951](https://github.com/n8n-io/n8n/issues/43951) · **Module:** `source-control.ee` · **Status:** Merged 2026-11-25

## Problem

`SourceControlImportService` reads credential and project export files from the work folder with a bare `jsonParse`. One malformed file — a partial write, a bad manual edit, an interrupted sync — makes *every* status call, push, and pull fail with a parse error that does not even name the offending file. The user has to hunt it down by hand.

## Root cause

Missing batch-failure isolation: the parse call sits in a loop with no per-item try/catch. The tell was that the *same service* already had the correct pattern for data tables — parse wrapped in try/catch, warn with the file path, skip the entry. Credentials and projects simply never got the treatment.

## Reproduction

1. Push once so the work folder contains `credential_stubs/*.json` and project files.
2. Corrupt one file's JSON (remove a closing brace).
3. Call source-control status / run a pull.
4. Before: the whole operation fails with a parse error naming nothing. After the fix: the good files flow; the log names the broken one.

## Fix approach

Give credentials and projects the data-tables treatment:

- in `getRemoteCredentialsFromFiles`, the batch callback catches parse failures, warns with the file path, and returns `undefined`; the downstream `flatMap`/`filter` already drop falsy entries;
- `getRemoteProjectsFromFiles` gets the same pattern plus an explicit skip-filter before access filtering.

Tests: a malformed credential file and a malformed project file are skipped with a warning naming the file, while remaining files still parse.

## Lessons

- **Sibling code is the spec.** When the same service guards one resource type but not another, that asymmetry is the bug.
- This fix also became the anchor for a follow-up ([case 4](04-credential-import-skip-unparsable.md)) — status-side and import-side readers needed the same guard separately, and the first PR made the remaining gap *visible*.

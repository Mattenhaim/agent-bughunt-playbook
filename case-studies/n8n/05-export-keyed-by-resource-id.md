# Case 5 — Export reports file entries by name while files are keyed by id

**PR:** [#44751](https://github.com/n8n-io/n8n/pull/44751) · **Issue:** [#44759](https://github.com/n8n-io/n8n/issues/44759) · **Module:** `source-control.ee` · **Status:** Open

## Problem

`ExportResult.files[].name` was built from the resource *name* (`getWorkflowPath(workflow.name)`, `` `${credential.name}.json` ``) while the file actually written to disk is keyed by the resource **id**. Every reported entry contradicted the files the export produced.

## Root cause

Id-vs-name confusion in a *reporting* path. Notably: no current caller consumed `files[]` — the push flow uses `missingIds`, which already keys by id — so there was **no behavioral bug today**, but every future consumer would receive wrong paths, and the reported entries contradicted reality.

## Reproduction

Export a workflow and a credential, then compare `ExportResult.files[0].name` against the files in the work folder: the reported name ends with `<resource-name>.json`, the file is `<resource-id>.json`. (Verified by test rather than UI — the API contract was the surface.)

## Fix approach

Report the path that is actually written: `getWorkflowPath(workflow.id)` and `getCredentialsPath(credential.id)`. No caller behavior changes. The workflow and credential export tests were extended to assert `files[0].name` ends with `<resource-id>.json`.

## Lessons

- **Honest scoping** matters: this was filed as a consistency bug, not a runtime failure, and the PR says explicitly "no behavior change for callers". Reviewers trust reports that don't inflate severity.
- Id-vs-name drift tends to live in *reporting* layers, where nothing breaks loudly enough to be noticed — grep for the mismatch between what is written and what is returned.

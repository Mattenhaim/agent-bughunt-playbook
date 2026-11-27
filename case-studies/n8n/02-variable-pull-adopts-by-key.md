# Case 2 — Variable pull blanks the local value on same-key adoption

**PR:** [#44748](https://github.com/n8n-io/n8n/pull/44748) · **Issue:** [#44757](https://github.com/n8n-io/n8n/issues/44757) · **Module:** `source-control.ee` · **Status:** Merged 2026-11-27

## Problem

When Git tracks a variable under a **different id** with the **same key** as one that exists locally (both instances created it independently before being connected), a pull empties the local variable's value. A forced pull compounds it: status also marks the local id deleted, so the variable is deleted and recreated empty.

## Root cause

A variable's real value is never synced — the exported stub carries `value: ''`. An earlier fix (#38488) had already made the *same-id* path leave existing values untouched. But the key-adoption path hits the key unique constraint and falls back to `update({ key }, { ...variable })` — which still sends the placeholder `''` and overwrites the stored value. Two code paths, one invariant, only one of them aware of the placeholder.

## Reproduction

1. Create a project variable `VAR1` with a real value.
2. In the repository's `variable_stubs.json`, define the same key under a different id with `value: ""`.
3. Force-pull.
4. Before: local value lost (variable recreated empty under the remote id). After: the variable adopts the remote id but keeps its local value.

## Fix approach

In the fallback, send `value: undefined` when the incoming value is the placeholder, so the ORM skips the column — the same rule the same-id path already follows. The test drives the upsert into a unique-violation error to reach the fallback deterministically.

## Lessons

- **Placeholder-aware writes** are an invariant: every write path must either skip the placeholder or be proved not to receive it.
- The failing test had to *force* the constraint violation — deterministic reproduction of an error path is worth the setup.

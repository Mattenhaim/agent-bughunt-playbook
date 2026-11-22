# Case 6 — Moving a variable into a scope that already has the same key creates a duplicate

**PR:** [#44771](https://github.com/n8n-io/n8n/pull/44771) · **Issue:** [#44774](https://github.com/n8n-io/n8n/issues/44774) · **Module:** variables (EE) · **Status:** Open

## Problem

`PATCH /variables/:id` with only `projectId` (no `key`) moves a variable into a project (or to global) that already holds a variable with the same key. The duplicate persists, and `$vars` for that scope becomes order-dependent: the runtime resolver lets the later row overwrite the earlier one, so workflows read a value that depends on database/cache ordering.

## Root cause

The update path only validated key uniqueness when the request *changed the key*. The create path and the rename path both validated; the move path didn't. The database has no `(key, project)` uniqueness constraint either, so nothing downstream caught it. The clincher: two precedence implementations in the codebase disagree about duplicate keys (last-write-wins vs first-match-wins) — evidence the duplicate state was never intended.

## Reproduction

1. Create variable `SHARED_KEY` in project A; create variable `SHARED_KEY` in project B (legal — different scopes).
2. `PATCH /variables/:id-of-A` with body `{"projectId": "<B>"}` — no `key` field.
3. Before: request succeeds; project B now has two `SHARED_KEY` variables. After: rejected with the same message the create path uses.

## Fix approach

In the project-change branch, validate the key that *will land* in the destination:

```
variable.key ?? existingVariable.key
```

so move-only validates the existing key and move+rename validates the new one (renaming away from a colliding key still works — no over-blocking). Tests: two rejection cases (destination project, destination global), one positive control, one move+rename case.

## Lessons

- **Check-then-act gaps hide in request-shape branches.** "The body has a key → validate; else skip" is a classic.
- The **two-implementations-disagree** signal was found by reading, and the duplicate state it implies was reachable only through this move path — reading found the door, the failing test opened it.
- No DB constraint existed either; the minimal fix stays at the service layer, where the project already validates.

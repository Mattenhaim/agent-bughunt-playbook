# Case 3 — Pull deletes a variable another file in the same pull moved

**PR:** [#44749](https://github.com/n8n-io/n8n/pull/44749) · **Issue:** [#44758](https://github.com/n8n-io/n8n/issues/44758) · **Module:** `source-control.ee` · **Status:** Merged 2026-11-27

## Problem

When one pull imports several team-project files, a variable that legitimately *moved* from project A to project B gets deleted outright — including its instance-local value — depending on file processing order.

## Root cause

`importTeamProjectsFromWorkFolder` pruned variables against a single `getAllCached()` snapshot taken **before** the loop. The earlier project's import legitimately moves a variable to another project; the stale snapshot still attributes it to the old project, so when the old project's file arrives without that stub, the just-imported variable looks orphaned and is pruned. A check-then-act gap with an ordering-dependent outcome.

## Reproduction

1. Source control with two team projects A and B; project A has a project variable.
2. In the repo's `projects/` files, move the variable stub from `A.json` to `B.json`.
3. Pull with the B file processed before A.
4. Before: the variable disappears entirely. After: it survives under project B.

## Fix approach

Read each project's variables from the database **right after** that project's import, instead of pruning against the pre-loop snapshot. The pruning semantics ("delete variables the imported project no longer has") are preserved; only the deletion list now reflects post-import ownership. The existing "delete variables not in the imported stubs" test keeps its intent with updated stubbing.

## Lessons

- **Snapshots across mutating loops** are a recurring bug shape — every loop iteration that can mutate the cached domain invalidates the snapshot.
- The bug needed *ordering* to manifest: the reproduction had to control file processing order, which the test achieved by stubbing the import sequence.

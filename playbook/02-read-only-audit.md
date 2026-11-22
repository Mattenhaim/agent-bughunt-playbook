# Step 2 — The read-only audit

**Goal:** find candidate bugs by *reading*, with zero side effects. No edits, no pushes, no posts.

## Run the agent read-only

Put the agent in plan/read-only mode. It should produce, per candidate:

- module, file, and line numbers;
- the invariant that is violated;
- a concrete reproduction sketch;
- a proposed minimal test.

Everything else — fixes, branches, commits — waits for later steps. Read-only mode also protects you from the classic failure where the agent "fixes" what it set out to study and you lose the evidence trail.

## Read for violated invariants, not for style

Style nitpicks are noise. The patterns that actually pay off:

- **Check-then-act:** a uniqueness check that runs on one request shape but not another (the [variable-move bug](../case-studies/n8n/06-variable-move-duplicate-key.md) was exactly this: `update()` validated the key only when the body carried a `key` field).
- **Two implementations of one rule:** when `resolveVariables` applies "last write wins" while `pickVariableForProject` applies "first match wins", a duplicated key produces different behavior depending on the reader — strong evidence the state is unintended.
- **Assumed invariants on user input:** `"a, b".split(',')` without trim (the [CORS bug](../case-studies/n8n/08-webhook-cors-origin-trim.md) — the code only ever matched entry 0 of a space-separated list).
- **Id vs name confusion:** files written keyed by id but reported keyed by name ([PR #44751](https://github.com/n8n-io/n8n/pull/44751)).
- **One malformed item poisoning a batch:** a bare parse call with no try/catch where sibling code has one ([PR #43952](https://github.com/n8n-io/n8n/pull/43952) — the data-tables reader already had the guard; credentials/projects didn't).
- **Stale snapshots across a loop:** a cache read before a loop that mutates what the cache describes ([PR #44749](https://github.com/n8n-io/n8n/pull/44749)).

## Trace claims to the actual layer

Every suspicion must be traced across the caller→implementation boundary before it is reported. Example from the n8n round 2: an apparent "pathLength off-by-one" in webhook routing dissolved after reading the node-side path construction (`isFullPath` flags) — the two sides were consistent. Recording a disproven suspicion costs one paragraph; filing it costs maintainer trust.

## Keep a findings ledger

One entry per candidate: **suspected → traced → proven (failing test) → disproven**, with file/line and the next action. The ledger becomes the hunt report and, later, the PR bodies — no re-derivation at write-up time.

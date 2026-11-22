# Case 7 — Allowed Origins (CORS) ignored for webhooks with dynamic paths

**PR:** [#44772](https://github.com/n8n-io/n8n/pull/44772) · **Issue:** [#44775](https://github.com/n8n-io/n8n/issues/44775) · **Module:** webhooks · **Status:** Open

## Problem

For a Webhook node whose path contains dynamic parameters (`user/:id`), the node's Allowed Origins (CORS) setting is silently ignored: the CORS handler falls back to reflecting whatever `Origin` the caller sends, so any third-party page can read the webhook's response cross-origin. Static paths enforce the list correctly.

## Root cause

`findAccessControlOptions` matched nodes by comparing the node's configured path with the concrete request path — which can never be equal for a dynamic webhook: the template `user/:id` registers under `<webhookId>/user/:id`, the request arrives as `<webhookId>/user/123`. Only the Chat Trigger had an explicit branch for its fixed suffix; user-defined dynamic paths were simply unreachable. The test-webhook manager had the same defect in a different costume: its registration-key search used `key.includes(path)`, which for dynamic keys (`method|webhookId|segmentCount`) can never match the concrete path — and which could also resolve a *shorter* request path to the *wrong* registration via substring matching.

## Reproduction

1. Webhook node, path `user/:id`, options Allowed Origins = `https://a.com`.
2. Activate; call `/webhook/<webhookId>/user/123` with `Origin: https://b.com`.
3. Before: `Access-Control-Allow-Origin: https://b.com` (reflected — the configured list ignored). After: requests from listed origins get the echo; others get the first allowed entry.

## Fix approach

- **Live-webhooks:** add a match arm keyed on the webhook row's `webhookId` — only dynamic registration rows carry it (static rows keep the column NULL), so it cannot collide with another node.
- **Test-webhooks:** replace the substring search with the same exact resolution the execution path already uses (`getRegistrationsFromPath`), then match the node by the registration's owning node name.
- Cubic's post-review also caught that the resolver's method list lacked `HEAD` — fixed in a follow-up commit with a HEAD test (fails before, passes after) plus regression tests for the removed substring matching.

Tests: dynamic-path lookup (fails on master, both managers), static-path positive control, HEAD registration, substring-embed exact-match, no-match → undefined.

## Lessons

- **Security-adjacent settings deserve a path-type matrix**: static × dynamic × method — the bug lived in exactly the cell nobody had tested.
- The AI reviewer's catch was real (P2, confidence 8) and cheap to fix once verified with the stash trick. Triage bot comments, verify, fix with tests; never argue in the thread.
- One defect can exist in two sibling implementations with different costumes — search the *interface*, not just the file.

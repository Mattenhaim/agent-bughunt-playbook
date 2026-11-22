# Case 8 — CORS entries after the first break when the list contains spaces

**PR:** [#44773](https://github.com/n8n-io/n8n/pull/44773) · **Issue:** [#44776](https://github.com/n8n-io/n8n/issues/44776) · **Module:** webhooks · **Status:** Open

## Problem

The webhook Allowed Origins (CORS) list is split on commas without trimming whitespace. With the common `https://a.com, https://b.com` spelling, every entry after the first keeps a leading space and never matches the request's `Origin` header — the response carries the first entry instead, and the browser rejects it. Only the first entry of a space-separated list works.

## Root cause

One line: `allowedOrigins.split(',')` with no trim, then `originsList.includes(req.headers.origin)`. The second entry is `' https://b.com'`; the comparison fails; the code falls back to `defaultOrigin` (entry 0).

## Reproduction

1. Webhook node, static path, Allowed Origins = `https://a.com, https://b.com` (comma + space).
2. Call the webhook with `Origin: https://b.com`.
3. Before: response header is `https://a.com` — browser CORS failure despite the origin being explicitly allowed. After: `https://b.com` is echoed.
4. Control: without the space (`https://a.com,https://b.com`) the second origin matches — which is exactly why the bug hides.

## The dedup story — a closed issue that wasn't fixed

Issue [#18528](https://github.com/n8n-io/n8n/issues/18528) reported precisely this symptom in August 2025 and was closed as "verified fixed in 1.109.0". Before filing a duplicate, the closure was audited: the handler source at tag `n8n@1.109.0` was fetched via the public API and is byte-identical to current master — no trim ever landed. A minimal failing test on master confirmed it. The new issue states the audit factually (what was claimed, what was checked, what still reproduces) instead of arguing.

## Fix approach

```
allowedOrigins.split(',').map(o => o.trim()).filter(o => o.length > 0)
```

plus a comment explaining why (users type spaces). Tests: preflight OPTIONS and regular GET with a whitespace-separated list — both fail on master, pass after.

## Lessons

- **Verify closed-issue claims against the tagged source** — closures get verified against a re-typed config, not against the code.
- The control step (no-space works) is what makes this reproducible on any machine and isolates whitespace as the cause.
- The whole fix is one line; the value is in the failing tests and the honest dedup audit.

# Prompt: anonymous dedup search

Run before opening any issue or PR. Read-only against GitHub's public API/web — no authenticated calls needed for public repos, and no comments anywhere.

```text
Check whether this bug is already reported in <owner>/<repo>. Use ONLY anonymous, read-only
access (public web search / unauthenticated API). No tokens, no comments, no issue edits.

Bug summary: <one paragraph>
Key identifiers: <exact error strings, function/class names, flag names>

1) Search open and closed issues AND pull requests for:
   - exact error strings (quoted);
   - function/class/flag names combined with symptom words;
   - the issue numbers referenced by any related PR the search surfaces.
2) For each plausible hit, read it and classify: same root cause? same symptom, different cause?
   fixed already (which version)? open as of today?
3) Report a table: hit — relation (duplicate / related / prior fix / unrelated) — evidence line.
4) If a closed issue claims a fix, verify the claim: read the code at the version tag named in
   the closure and check whether the parsing/branch in question is actually unchanged
   (cite the file and commit/tag you checked).
5) Conclusion: new report / revive with reference / skip.
```

## Notes

- The **verify-the-closure** step is how we found that a real CORS bug sat behind an issue closed as "fixed in 1.109.0" — the tagged source was byte-identical to master and still broken. One factual sentence in the new issue then does the arguing for you.
- Anonymous access avoids touching any credentials and keeps the action scope clean; public search endpoints are sufficient.
- Search both issues and PRs — prior *PRs* often contain the root-cause analysis even when their issue is thin.

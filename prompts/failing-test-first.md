# Prompt: failing-test-first proof

Use after the audit produces a candidate. The agent is allowed to edit the repo now — but only to add tests.

```text
You are on branch <branch> of <project> at <base-commit>.

Prove this bug with tests before touching any implementation:

  Candidate: <one-paragraph description, file:line, invariant violated>
  Reproduction sketch: <from the audit>

1) Read the existing tests next to <relevant-path> and reuse their idiom
   (mock builders, env setup, assertion style). Do not invent a new harness.
2) Write the minimal test(s) that fail on the CURRENT checkout (which equals upstream master):
   - one test per broken contract, asserting observable behavior (error message, returned value,
     written rows), not internals;
   - one positive control: the same operation against a clean scope must still succeed
     (this guards the future fix against over-blocking);
   - name tests after the failure mode ("rejects the move when the destination project
     already has the same key").
3) Run them. Expected: the bug test(s) FAIL, the positive control PASSES.
   If everything passes, the bug is not real — say so and stop.
4) Report: exact commands to run, which tests fail and with what output, and the diff stat
   (test files only — no implementation changes yet).
```

## Notes

- The **positive control** is what makes the later fix trustworthy: it shows the new validation isn't a blanket refusal.
- **"If everything passes, the bug is not real"** gives the agent an explicit exit — the single best defense against rationalizing a phantom bug into a test that always passes.
- Keep the test diff separate from the fix diff: reviewers read them differently, and the stash trick (run new tests against unfixed implementation) needs the split.

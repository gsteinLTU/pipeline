---
name: tdd
description: Test-driven development at pre-agreed seams. Use when the user wants to build features or fix bugs test-first, mentions "red-green-refactor", or wants integration tests. Not for deciding where the seam goes — use codebase-design for that; not for the per-ticket or whole-branch review that follows the refactor pass — use subagent-execution or code-review for that.
---

# Test-Driven Development

TDD is the red → verify red → green → verify green loop, plus one refactor pass at the end of each ticket. This skill is the reference that makes that loop produce tests worth keeping: what a good test is, where tests go, the anti-patterns, and the rules of the loop and the Iron Law that gives it teeth. Every section applies on every cycle: consult them before and during the loop, not after.

When exploring the codebase, read `CONTEXT.md` (if it exists) so test names and interface vocabulary match the project's domain language, and respect ADRs in the area you're touching.

## The Iron Law

```
NO PRODUCTION CODE WITHOUT A FAILING TEST FIRST
```

Write code before the test? Delete it. Start over.

**No exceptions:**
- Don't keep it as "reference"
- Don't "adapt" it while writing tests
- Don't look at it
- Delete means delete

Implement fresh from tests. Period.

Thinking "skip TDD just this once"? Stop. That's rationalization. **Violating the letter of the rules is violating the spirit of the rules.**

| Excuse | Reality |
|---|---|
| "Keep as reference, write tests first" | You'll adapt it. That's testing after. Delete means delete. |
| "Deleting X hours is wasteful" | Sunk cost fallacy — that time is already spent either way. The real choice: rewrite with TDD (high confidence) vs. keep it and bolt tests on after (low confidence, likely bugs). Keeping code you can't trust is the waste. |
| "Tests after achieve the same goals" | Tests-after answer "what does this do?"; tests-first answer "what should this do?" You never watched it fail, so you never proved it can catch the bug. |
| "Already manually tested" | Manual testing is ad-hoc: no record of what you covered, no way to re-run it when the code changes. |

## What a good test is

Tests verify behavior through public interfaces, not implementation details. Code can change entirely; tests shouldn't. A good test reads like a specification: "user can checkout with valid cart" tells you exactly what capability exists, and it survives refactors because it doesn't care about internal structure.

See [tests.md](tests.md) for examples and [mocking.md](mocking.md) for mocking guidelines.

## Seams: where tests go

A **seam** is the public boundary you test at: the interface where you observe behavior without reaching inside. Tests live at seams, never against internals.

**Test only at pre-agreed seams.** Before writing any test, write down the seams under test and confirm them with the user. No test is written at an unconfirmed seam. You can't test everything, so agreeing the seams up front is how testing effort lands on the critical paths and complex logic instead of every edge case.

Ask: "What's the public interface, and which seams should we test?"

When the shape of that interface is itself in question (how deep the module is, where the seam belongs, what the interface should expose), call the Skill tool with "pipeline:codebase-design" for the vocabulary. It is the shared source of the module, interface, depth, seam, adapter, leverage and locality terms, and it is a reference to consult, not a session to run.

## Anti-patterns

- **Implementation-coupled**: mocks internal collaborators, tests private methods, or verifies through a side channel (querying the database instead of using the interface). The tell: the test breaks when you refactor but behavior hasn't changed.
- **Tautological**: the assertion recomputes the expected value the way the code does (`expect(add(a, b)).toBe(a + b)`, a snapshot derived by hand the same way, a constant asserted equal to itself), so it passes by construction and can never disagree with the code. Expected values must come from an independent source of truth: a known-good literal, a worked example, the spec.
- **Horizontal slicing**: writing all tests first, then all implementation. Bulk tests verify _imagined_ behavior: you test the _shape_ of things rather than user-facing behavior, the tests go insensitive to real changes, and you commit to test structure before understanding the implementation. Work in **vertical slices** instead: one test → one implementation → repeat, each test a **tracer bullet** that responds to what the last cycle taught you.

## The loop

```
RED (write failing test) → verify RED (mandatory) → GREEN (minimal code) → verify GREEN (mandatory) → repeat
```

- **Red before green.** Write the failing test first, then only enough code to pass it. Don't anticipate future tests or add speculative features.
- **One slice at a time.** One seam, one test, one minimal implementation per cycle.

### RED — Write Failing Test

Write one minimal test showing what should happen: one behavior, a clear name, real code (no mocks unless unavoidable).

### Verify RED — Watch It Fail

**MANDATORY. Never skip.** Run the test. Confirm it fails (not errors), the failure message is expected, and it fails because the feature is missing, not because of a typo. Test passes? You're testing existing behavior — fix the test. Test errors? Fix the error, re-run until it fails correctly.

### GREEN — Minimal Code

Write the simplest code that passes the test. Don't add features, refactor other code, or "improve" beyond the test — that belongs to the refactor pass, not this step.

### Verify GREEN — Watch It Pass

**MANDATORY.** Run the test. Confirm it passes, other tests still pass, and the output is pristine (no stray errors or warnings). Test fails? Fix the code, not the test. Other tests fail? Fix now.

## Refactoring is per ticket, not per cycle

Refactoring does not happen inside the red/green loop — don't clean up mid-cycle, and don't add behavior while refactoring. Instead, once every acceptance criterion on the current ticket is green, do **one refactor pass** for the whole ticket: remove duplication, improve names, extract helpers, keeping every test green throughout. That pass is the last thing that happens before the ticket goes to review (see `subagent-execution`), so review sees refactored code rather than being asked to request refactoring separately.

## Common rationalizations

| Excuse | Reality |
|---|---|
| "Too simple to test" | Simple code breaks. Test takes 30 seconds. |
| "Need to explore first" | Fine. Throw away exploration, start with TDD. |
| "Test hard = design unclear" | Listen to the test. Hard to test = hard to use. |
| "TDD will slow me down" | TDD IS the pragmatic path: catches bugs before commit, prevents regressions, lets you refactor without fear. |
| "Existing code has no tests" | You're improving it. Add tests for existing code. |

When writing or changing any test, read [tests.md](tests.md) for the rules that keep tests honest: name the production change that would make the test fail before writing it; assert on real behavior, never on mock behavior; keep test-only code in test utilities, out of production classes.

## Verification checklist

Before marking a ticket's tests complete:

- [ ] Every new function/method has a test
- [ ] Watched each test fail before implementing
- [ ] Each test failed for the expected reason (feature missing, not typo)
- [ ] Wrote minimal code to pass each test
- [ ] All tests pass
- [ ] Output pristine (no errors, warnings)
- [ ] Tests use real code (mocks only if unavoidable)
- [ ] Edge cases and errors covered
- [ ] The end-of-ticket refactor pass is done and tests are still green

Can't check all boxes? You skipped a step. Go back and do it properly, don't just note the gap.

## Debugging integration

Bug found? Write a failing test reproducing it, follow the loop above. The test proves the fix and prevents regression. Never fix bugs without a test — if the bug is hard to pin down first, that's `diagnosing-bugs`' job, not this skill's.

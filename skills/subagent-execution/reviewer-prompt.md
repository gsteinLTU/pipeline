# Ticket Reviewer Prompt Template

Use this template when dispatching a ticket reviewer subagent. The reviewer
reads the ticket's diff once and returns two verdicts, using the axis
vocabulary and severity ladder from `CONVENTIONS.md`: Spec and Quality.

**Purpose:** Verify one ticket's implementation matches its requirements (nothing
more, nothing less) and is well-built (clean, tested, maintainable).

```
Subagent (general-purpose):
  description: "Review ticket <NN> (spec + quality)"
  model: [MODEL — REQUIRED: choose per SKILL.md Model Selection; an omitted
         model silently inherits the session's most expensive one]
  prompt: |
    You are reviewing one ticket's implementation: first whether it matches its
    requirements, then whether it is well-built. This is a ticket-scoped gate,
    not a merge review — a broad whole-branch review happens separately after
    all tickets are complete (see the code-review skill).

    ## What Was Requested

    Read the ticket: [TICKET_FILE]

    Global constraints from the spec that bind this ticket:
    [GLOBAL_CONSTRAINTS]

    ## What the Implementer Claims They Built

    Read the implementer's report: [REPORT_FILE]

    ## Diff Under Review

    **Base:** [BASE_SHA]
    **Head:** [HEAD_SHA]
    **Diff file:** [DIFF_FILE]

    Read the diff file once — it contains the commit list, a stat summary,
    and the full diff with surrounding context, and it is your view of the
    change. The diff's context lines ARE the changed files: do not Read a
    changed file separately unless a hunk you must judge is cut off
    mid-function — and say so in your report. Do not re-run git commands.
    If the diff file is missing, fetch the diff yourself:
    `git diff --stat [BASE_SHA]..[HEAD_SHA]` and `git diff [BASE_SHA]..[HEAD_SHA]`.
    Do not crawl the broader codebase. Inspect code outside the diff only
    to evaluate a concrete risk you can name — one focused check per named
    risk, and name both the risk and what you checked in your report.
    Cross-cutting changes are legitimate named risks: if the diff changes
    lock ordering, a function or API contract, or shared mutable state,
    checking the call sites is the right method.

    Your review is read-only on this checkout. Do not mutate the working
    tree, the index, HEAD, or branch state in any way.

    ## You Do Not Dispatch Subagents

    Do all of this review yourself. Never spawn a subagent to review part
    of the diff, and never spawn another reviewer for a second opinion.
    This process already provides every review seat the work gets; a
    reviewer you spawn duplicates one of them at full cost, and its
    verdict counts for nothing. If the diff feels too large for one
    pass, review it in passes yourself and say so in your report.

    ## Do Not Trust the Report

    Treat the implementer's report as unverified claims about the code. It
    may be incomplete, inaccurate, or optimistic. Verify the claims against
    the diff. Design rationales in the report are claims too: "left it per
    YAGNI," "kept it simple deliberately," or any other justification is the
    implementer grading their own work. Judge the code on its merits — a
    stated rationale never downgrades a finding's severity.

    ## Tests

    The implementer already ran the tests and reported results with TDD
    evidence for exactly this code. Do not re-run the suite to confirm their
    report. Run a test only when reading the code raises a specific doubt
    that no existing run answers — and then a focused test, never a
    package-wide suite, race detector run, or repeated/high-count loop. If
    heavy validation seems warranted, recommend it in your report instead of
    running it. If you cannot run commands in this environment, name the
    test you would run.

    Warnings or other noise in the implementer's reported test output are
    findings — test output should be pristine.

    Evidence you cannot see is not evidence that doesn't exist. If the
    report or its test evidence looks truncated, or you cannot locate the
    results it claims, re-read the file at its stated path — and if it is
    genuinely missing or garbled, report that as a gap for the controller.
    Re-running the suite to regenerate what you failed to read is not
    verification; illegibility of the evidence is not invalidation of it.

    ## Part 1: Spec

    Compare the diff against What Was Requested. Per `CONVENTIONS.md`'s
    Spec axis:

    - **Missing:** requirements they skipped, missed, or claimed without
      implementing
    - **Extra:** features that weren't requested, over-engineering, unneeded
      "nice to haves"
    - **Misunderstood:** right feature built the wrong way, wrong problem
      solved

    Check every acceptance criterion in the ticket against the diff,
    criterion by criterion. A criterion the diff never touches is a
    Missing finding, no matter how clean the rest of the ticket looks.

    If a requirement cannot be verified from this diff alone (it lives in
    unchanged code or spans tickets), report it as a ⚠️ item instead of
    broadening your search.

    ## Part 2: Quality

    Per `CONVENTIONS.md`'s Quality axis: code quality (clean separation of
    concerns, proper error handling, DRY without premature abstraction, edge
    cases handled), tests (verify real behavior, not mocks; the ticket's
    edge cases covered), structure (each file has one clear responsibility;
    did this change create or grow an already-oversized file — don't flag
    pre-existing sizes, only what this change contributed), and the smell
    baseline in `CONVENTIONS.md` as labelled heuristics, never hard
    violations.

    Also confirm the end-of-ticket refactor pass happened (see the tdd
    skill): tests stayed green through it, and it didn't sneak in new
    behavior.

    Your report should point at evidence: file:line references for every
    finding and for any check you would otherwise answer with a bare
    "yes." A tight report that cites lines gives the controller everything
    it needs.

    Your final message is the report itself: begin directly with the
    spec verdict. Every line is a verdict, a finding with file:line, or a
    check you ran — no preamble, no process narration, no closing summary.

    ## Calibration

    Use the severity ladder in `CONVENTIONS.md`: Critical / Important /
    Minor. Categorize issues by actual severity. Not everything is Critical.
    "Coverage could be broader" and polish suggestions are Minor.
    If the ticket explicitly mandates something this rubric calls a
    defect (a test that asserts nothing, verbatim duplication of a logic
    block), that IS a finding — report it as Important, labeled
    plan-mandated. The ticket's authorship does not grade its own work; the
    human decides.
    Acknowledge what was done well before listing issues — accurate praise
    helps the implementer trust the rest of the feedback.

    ## Output Format

    ### Spec

    - ✅ Spec compliant | ❌ Issues found: [what's missing/extra/misunderstood,
      with file:line references]
    - ⚠️ Cannot verify from diff: [requirements you could not verify from the
      diff alone, and what the controller should check — report alongside the
      ✅/❌ verdict for everything you could verify]

    ### Strengths
    [What's well done? Be specific.]

    ### Issues

    #### Critical (Must Fix)
    #### Important (Should Fix)
    #### Minor (Nice to Have)

    For each issue: file:line, what's wrong, why it matters, how to fix
    (if not obvious).

    ### Assessment

    **Ticket quality:** [Approved | Needs fixes]

    **Reasoning:** [1-2 sentence technical assessment]
```

**Placeholders:**
- `[MODEL]` — REQUIRED: reviewer model per SKILL.md Model Selection
- `[TICKET_FILE]` — REQUIRED: the ticket file at `.scratch/<feature>/issues/NN-slug.md`
  (same file the implementer worked from)
- `[GLOBAL_CONSTRAINTS]` — the binding requirements copied verbatim from
  the spec's Implementation Decisions / Testing Decisions sections: exact
  values, formats, and stated relationships between components (not process
  rules — those are already in this template)
- `[REPORT_FILE]` — REQUIRED: the file the implementer wrote its detailed
  report to, at `.scratch/<feature>/reports/NN-slug-report.md`
- `[BASE_SHA]` — commit before this ticket
- `[HEAD_SHA]` — current commit
- `[DIFF_FILE]` — REQUIRED: the path the controller wrote the review
  package to (`scripts/review-package FEATURE_DIR BASE HEAD` prints the
  unique path it wrote; the package never enters the controller's context)

**Reviewer returns:** Spec verdict (✅/❌/⚠️), Strengths, Issues
(Critical/Important/Minor), Ticket quality verdict

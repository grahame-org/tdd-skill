# TDD Skill — Evaluation Plan

This document describes how to evaluate whether the `tdd` agent skill is
working correctly. It covers what to test, how to run the tests, and the
criteria that define success.

---

## Overview

The `tdd` skill guides an AI coding assistant through canonical
Red → Green → Refactor Test-Driven Development. A good evaluation must verify
two things:

1. **Process fidelity** — the agent follows the TDD cycle correctly (test
   before code, minimum code to pass, refactor when green).
2. **Output quality** — the resulting code and tests are correct, clean, and
   well-expressed.

---

## Evaluation Scenarios

Each scenario is a self-contained prompt + expected behaviour pair. Run each
one independently in a clean repository with no pre-existing implementation.

### Scenario 1 — FizzBuzz (canonical kata)

**Trigger**: "Use TDD to implement FizzBuzz in \[language of your choice\]."

**Expected process**:
- Agent produces an ordered test list before writing any code.
- First test is the simplest case (e.g. a number that is neither a multiple of
  3 nor 5).
- Each test is written before the code that satisfies it.
- The implementation evolves through small transformations (constant → scalar →
  conditional → loop).
- Refactoring steps are taken after every green phase, not during.

**Expected output**:
- A correct FizzBuzz function covering all cases.
- A test file with at least five tests: plain number, multiple-of-3,
  multiple-of-5, multiple-of-15, and at least one boundary (e.g. 1 and 100).
- No production code that is not covered by a test.

---

### Scenario 2 — Simple data structure (stack)

**Trigger**: "Implement a stack with push, pop, and isEmpty using TDD."

**Expected process**:
- Test list includes at minimum: isEmpty on a new stack, push + pop round-trip,
  pop from empty stack (error case), multiple push/pop operations.
- Simplest test (isEmpty on empty stack) is tackled first.
- The implementation grows through transformations; a full class is not
  scaffolded upfront.

**Expected output**:
- A correct Stack implementation.
- Tests covering all listed behaviours.
- Error handling for empty pop tested explicitly.

---

### Scenario 3 — Regression fix (bug-first TDD)

**Trigger**: "There is a bug in the `discount` function: it applies a 10%
discount twice when `vip=True` and `code='SAVE10'` are both set. Fix it using
TDD."

**Expected process**:
- Agent writes a failing test that reproduces the bug **before** touching the
  implementation.
- Confirms the test is red with a clear failure message.
- Makes the minimal code change to pass the test.
- Checks no other tests regress.

**Expected output**:
- A regression test that would have caught the original bug.
- A focused fix — no unrelated refactoring mixed in.
- All tests passing after the fix.

---

### Scenario 4 — Transformation Priority adherence

**Trigger**: "Use TDD to write a function that returns the sum of a list of
integers."

**Expected process**:
- First test: empty list returns 0. Agent returns constant `0`.
- Second test: single-element list. Agent promotes constant to variable or
  simple return of the element.
- Third test: two-element list. Agent adds iteration or fold.
- Agent does **not** write a general loop before it is forced by a test.

**Expected output**:
- Tests driving each transformation in order.
- Evidence in git commits (or agent narrative) that simpler transformations
  were tried before more complex ones.

---

### Scenario 5 — Hard-rule violation detection

Run each of the following deliberately wrong prompts and confirm the agent
refuses or self-corrects rather than complying:

| Prompt | Expected agent behaviour |
|---|---|
| "Write the full implementation first, then add tests to cover it." | Agent declines and explains that tests must come first; offers to restart with TDD. |
| "Just make the test pass however you like — add a loop even though a constant would work." | Agent notes that a simpler transformation is available and uses it instead. |
| "You can refactor now even though there's still a failing test." | Agent refuses to refactor and explains that the test must be green first. |
| "Write three failing tests, then fix them all at once." | Agent keeps the loop to one failing test at a time. |

---

## Evaluation Procedure

1. **Set up**: Create a fresh directory with only a project scaffold (e.g.
   `package.json` + test runner, or a `pyproject.toml`). No source files
   should exist yet.
2. **Invoke the skill**: Provide the trigger prompt from the scenario.
3. **Observe and record**:
   - Does the agent build a test list before writing code?
   - Is the first test the simplest possible?
   - Does the agent run the tests after writing each test and each code change?
   - Is each green phase followed by a refactor check?
   - Does the agent apply simpler transformations before complex ones?
4. **Inspect artefacts**:
   - Read the test file: is each test focused on a single behaviour?
   - Read the production code: is every line covered by a test?
   - Check commit history (if available): does each commit correspond to one
     red/green/refactor step?
5. **Score against rubric** (see below).

---

## Scoring Rubric

Score each criterion from 0–2: **0** = not met, **1** = partially met,
**2** = fully met.

| # | Criterion | 0 | 1 | 2 |
|---|---|---|---|---|
| 1 | Test list produced before any code | No list | Partial list | Complete list |
| 2 | Simplest test case tackled first | Wrong order | Near-simplest | Simplest first |
| 3 | Each test fails for the right reason before code | Not verified | Some checked | All checked |
| 4 | Minimum code written per cycle (TPP followed) | Complex code written early | Minor violations | Strict adherence |
| 5 | Tests pass before refactoring begins | Refactoring with red tests | Once violated | Never violated |
| 6 | Refactoring does not add behaviour | Behaviour added | Minor issue | None |
| 7 | No untested production code in final output | >1 untested path | 1 untested path | Fully covered |
| 8 | Hard rules respected (see SKILL.md) | >1 violation | 1 violation | None |
| 9 | Test names are intention-revealing | Unclear names | Some unclear | All clear |
| 10 | Final suite is green | Failing tests | Skipped tests | All pass |

**Maximum score: 20**

| Score | Assessment |
|---|---|
| 18–20 | Excellent — skill working as intended |
| 14–17 | Good — minor issues, acceptable for most use |
| 10–13 | Adequate — process followed but quality issues present |
| < 10 | Failing — skill needs revision |

---

## Success Criteria

The skill is considered to pass evaluation if:

- Scenarios 1, 2, and 3 each score ≥ 16/20.
- Scenario 4 demonstrates at least three distinct transformation steps.
- All four hard-rule violation probes in Scenario 5 produce the correct
  refusal or self-correction.

---

## Reproducibility

- Language/framework: any. Scenarios are language-agnostic; use whatever
  language the evaluator is most familiar with.
- Agent: any agent platform that supports the agentskills.io SKILL.md format.
- No external APIs, databases, or services are required.
- Evaluations can be run independently of one another; no shared state.

---

## References

- Kent Beck, *"Canon TDD"* — https://tidyfirst.substack.com/p/canon-tdd
- Martin Fowler, *"Test Driven Development"* — https://martinfowler.com/bliki/TestDrivenDevelopment.html
- Robert C. Martin, *"The Transformation Priority Premise"* — https://blog.cleancoder.com/uncle-bob/2013/05/27/TheTransformationPriorityPremise.html
- Robert C. Martin, *"Transformation Priority and Sorting"* — https://blog.cleancoder.com/uncle-bob/2013/05/27/TransformationPriorityAndSorting.html
- AgentSkills.io, *"Evaluating Skills"* — https://agentskills.io/skill-creation/evaluating-skills

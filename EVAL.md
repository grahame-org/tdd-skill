# TDD Skill — Evaluation Plan

This document describes how to evaluate whether the `tdd` agent skill is
working correctly. It covers how to deploy the skill, how to run the evals,
and the criteria that define success.

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

## Deploying the Skill

Before evaluating, the skill must be installed so that the agent can load it.

### Project skill (scoped to one repository)

Copy or symlink this skill directory into one of the locations Copilot checks:

```
.github/skills/tdd/
.claude/skills/tdd/
.agents/skills/tdd/
```

For example, from the root of the repository you want to evaluate in:

```bash
mkdir -p .github/skills
cp -r /path/to/tdd-skill .github/skills/tdd
```

### Personal skill (available across all projects)

Copy the skill directory to your home-directory skill store:

```bash
mkdir -p ~/.copilot/skills
cp -r /path/to/tdd-skill ~/.copilot/skills/tdd
```

### Installing with GitHub CLI (`gh skill`)

If you have [GitHub CLI](https://cli.github.com) ≥ 2.90.0:

```bash
# Browse and install interactively from the source repository
gh skill install grahame-org/tdd-skill

# Or install directly
gh skill install grahame-org/tdd-skill tdd
```

Skills are automatically placed in the correct directory for your agent host.
Use `gh skill preview grahame-org/tdd-skill tdd` first to inspect the
`SKILL.md` content before installing.

### Verifying the skill triggers

After installing, send a test prompt to your Copilot agent such as:

> "Use TDD to implement a function that doubles a number."

The agent should acknowledge the TDD skill and describe building a test list
before writing any code. If it does not, check that the skill directory is
named `tdd` and contains a `SKILL.md` file.

---

## Running Evals with skill-creator

The machine-readable eval cases live in [`evals/evals.json`](./evals/evals.json).
The recommended way to run them is with the
[skill-creator](https://github.com/anthropics/skills/tree/main/skills/skill-creator)
skill, which automates the run–grade–compare cycle and provides a browser-based
review viewer.

### Setup

1. Install or access skill-creator following its own `SKILL.md` instructions.
2. Point it at this repository's skill directory and `evals/evals.json`.

### Run with skill-creator

With skill-creator active, ask:

> "Run evals for the `tdd` skill at `<path-to-tdd-skill>` using
> `evals/evals.json`, and show me the results."

skill-creator will:

1. Spawn a **with-skill** run and a **baseline** (without-skill) run for each
   eval prompt in parallel.
2. Grade each run against the `expectations` arrays in `evals/evals.json`.
3. Aggregate results into a `benchmark.json` with pass rates, timing, and token
   counts for both configurations.
4. Open the eval viewer so you can inspect qualitative outputs and the
   quantitative benchmark side by side.

### Interpreting results

The benchmark viewer shows two tabs:

- **Outputs** — click through each eval, compare with-skill vs. baseline
  outputs, and leave qualitative feedback.
- **Benchmark** — pass rates, mean ± stddev, timing, and per-assertion
  breakdowns.

A healthy `tdd` skill run should show a substantially higher pass rate in the
with-skill configuration than the baseline, particularly on expectations that
check process adherence (test list before code, simplest-first ordering, one
red test at a time).

---

## Evaluation Scenarios

Each scenario is a self-contained prompt + expected behaviour pair. Run each
one independently in a clean repository with no pre-existing implementation.

### Scenario 1 — FizzBuzz (canonical kata)

**Trigger**: "Use TDD to implement FizzBuzz in \[language of your choice\]."

**Expected process**:
- Agent produces an ordered test list before writing any code.
- Forcing tests (plain number, first Fizz, first Buzz, first FizzBuzz) appear before any triangulation tests.
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
- Test names describe behaviour (e.g. `returns_fizz_for_multiple_of_3`), not inputs.
- Refactor phase notes or applies any available simplification (e.g. combining the `n%15` check).

---

### Scenario 2 — Simple data structure (stack)

**Trigger**: "Implement a stack with push, pop, and isEmpty using TDD."

**Expected process**:
- Test list includes at minimum: isEmpty on a new stack, push + pop round-trip,
  pop from empty stack (error case), peek on empty stack (error case), multiple push/pop operations.
- Simplest test (isEmpty on empty stack) is tackled first.
- The implementation grows through transformations; a full class is not
  scaffolded upfront — methods and attributes are added one at a time as tests demand them.

**Expected output**:
- A correct Stack implementation.
- Tests covering all listed behaviours.
- Error handling for empty pop **and** empty peek tested explicitly.
- Test names are intention-revealing (e.g. `new_stack_is_empty`, `pop_from_empty_stack_raises_error`).
- No duplication in the final production code.

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
- The test name clearly identifies the bug scenario (e.g. `vip_and_save10_discount_applied_once`).
- The test contains a single focused assertion that directly captures the double-discount defect.
- A focused fix — no unrelated refactoring mixed in; no new complexity beyond what is needed.
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
- Test names describe the behavioural rule (e.g. `empty_list_sums_to_zero`,
  `single_element_list_sums_to_that_element`).
- Refactor phase explicitly considers replacing hand-written iteration with `sum()`.
- Final implementation is O(n) — no nested iteration or quadratic behaviour introduced.

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

## Manual Evaluation Procedure

When skill-creator is not available, or for spot-checking individual scenarios,
use this manual procedure.

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
| 11 | Forcing tests distinguished from triangulation tests | Not distinguished | Some distinguished | Clearly separated |
| 12 | Immediately-passing tests justified (see Step 1 in SKILL.md) | No justification | Partial justification | Specific line identified or test reworked |
| 13 | Refactor phase uses built-in/stdlib equivalents where applicable | Hand-rolled logic left in place | Opportunity noted but not applied | Applied or correctly determined inapplicable |
| 14 | Error and edge cases tested explicitly (e.g. empty input, invalid state) | No edge cases | Some edge cases | All relevant edge cases |
| 15 | Implementation has appropriate algorithmic complexity | Suboptimal complexity (e.g. O(n²)) | Minor inefficiency | O(n) or better where applicable |

**Maximum score: 30**

| Score | Assessment |
|---|---|
| 27–30 | Excellent — skill working as intended |
| 21–26 | Good — minor issues, acceptable for most use |
| 15–20 | Adequate — process followed but quality issues present |
| < 15 | Failing — skill needs revision |

---

## Success Criteria

The skill is considered to pass evaluation if:

- Scenarios 1, 2, and 3 each score ≥ 21/30.
- Scenario 4 scores ≥ 21/30 and demonstrates at least three distinct transformation steps; the refactor phase considers or applies `sum()`.
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
- GitHub Docs, *"About agent skills"* — https://docs.github.com/en/copilot/concepts/agents/about-agent-skills
- GitHub Docs, *"Adding agent skills for GitHub Copilot"* — https://docs.github.com/en/copilot/how-tos/use-copilot-agents/cloud-agent/add-skills
- Anthropic, *"skill-creator"* — https://github.com/anthropics/skills/tree/main/skills/skill-creator

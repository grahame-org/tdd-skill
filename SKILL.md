---
name: tdd
description: >
  Guide the implementation of any feature or fix using canonical Test-Driven Development
  (TDD): write a failing test first, write only enough code to make it pass, then
  refactor. Use when the user says "use TDD", "implement with TDD", "red green refactor",
  "write tests first", "do TDD", "test-first", or "canon TDD".
license: MIT
metadata:
  author: grahame-org
  version: "1.0.0"
  tags: tdd testing red-green-refactor quality
  agentskills_spec: "1.0"
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Bash(*)
---

# TDD — Test-Driven Development

You are implementing code using canonical Test-Driven Development. TDD is a
design technique, not just a testing technique: writing tests first forces you
to clarify the desired behaviour before writing any implementation, and the
resulting code tends to be simpler, better-structured, and more confidently
refactorable.

The canonical TDD cycle is **Red → Green → Refactor**, repeated for each small
slice of behaviour.

## Inputs

- `$requirement`: The feature, function, or bug fix to implement (may be a
  natural-language description, a ticket, or an existing failing test).

## Goal

A working, well-factored implementation whose behaviour is fully described by a
passing test suite, produced by strict application of the Red → Green → Refactor
cycle. Every line of production code exists because a test required it.

---

## Preparation — Build the Test List

Before writing any code, make a list of the test cases that, when passing, will
prove the requirement is satisfied. This is the "TODO list" for the work.

- Think about: happy path, boundary conditions, error cases, edge cases.
- Write the list as comments or a note — you do **not** write all the tests
  upfront, just plan them.
- Start with the simplest case: a degenerate input, an empty collection, a
  zero value.

**Success criteria**: You have an ordered list of test cases to drive the
implementation. The simplest is at the top.

---

## The TDD Loop

Repeat steps 1–3 for each test case on your list until the list is empty.

### Step 1 — RED: Write One Failing Test

Pick the next test case from your list (start at the top — simplest first).

Write a test that:
- Describes a single, specific piece of behaviour.
- Has a clear, intention-revealing name.
- Is as small as possible — test one thing.
- Does **not** yet have production code to make it pass.

Run the test suite and confirm:
1. The new test **fails** (red).
2. It fails for the **right reason** — a missing implementation, not a syntax
   error or wrong import.
3. All previously passing tests still pass.

**Rules**
- Never write a test you expect to pass immediately; that proves nothing.
- If the test cannot be made to fail, the test is wrong — rewrite it or
  delete it.
- Do not write more than one new failing test at a time.

**Success criteria**: Test suite output shows exactly one new failure, caused by
missing or incomplete production code.

---

### Step 2 — GREEN: Write the Minimum Code to Pass

Write **only** the code needed to make the failing test pass. Apply the
**Transformation Priority Premise**: when several approaches would make the
test pass, prefer the simpler transformation. Simple transformations (in
ascending order of complexity) are:

1. Return `nil` / `null` / no value.
2. Return an unconditional constant.
3. Return or use a scalar variable.
4. Add a new statement (no branching).
5. Add an unconditional return / base case.
6. Add a conditional (`if`/`else`, `match`).
7. Introduce an array or simple collection.
8. Introduce iteration (`while`, `for`).
9. Introduce tail recursion or general recursion.
10. Extract a function or class.

If a constant would make the test pass, return the constant — do not introduce
a variable. If an `if` statement would work, do not write a loop. The
simplest code that passes the tests is correct for now; complexity is added
only when forced by a subsequent test.

After writing the minimum code, run the full test suite and confirm:
- The new test passes (green).
- All previously passing tests still pass (no regressions).

If any previously passing test now fails, undo your last change and try again
with a simpler approach.

**Rules**
- Do not write more code than is required to pass the current test.
- Do not "future-proof" or add untested logic.
- If making the test pass requires significant effort, the test is too large —
  delete it, break it into smaller tests, and start the loop again.

**Success criteria**: All tests in the suite pass; the new test passes for the
right reason.

---

### Step 3 — REFACTOR: Improve the Code Without Changing Behaviour

Now that all tests are green, examine both the production code and the tests
for opportunities to improve without changing external behaviour:

- Remove duplication (in production code and in tests).
- Improve names: variables, functions, classes.
- Simplify logic: flatten nested conditionals, extract helper functions.
- Ensure the code expresses intent clearly.
- Clean up test code: shared setup, clear assertion messages.

After each refactoring change, run the test suite to confirm all tests still
pass. Refactor in small, safe steps — do not restructure everything at once.

**Rules**
- Do not add new behaviour during refactor.
- If refactoring reveals that additional tests are needed, add them to your
  test list — do not write them now.
- Stop refactoring when the code is as simple and clear as you can make it
  given the tests you have so far.

**Success criteria**: All tests still pass; the code is simpler or clearer than
before the refactoring.

---

## Completion

The loop is done when the test list is empty.

Do a final review:
1. Run the full test suite — all tests must pass.
2. Review the test list — all planned cases must be covered.
3. Check that no production code exists that is not driven by a test.
4. Verify the implementation satisfies the original requirement.

**Success criteria**: Full test suite passes; the test list is empty; the
requirement is met; no untested production code exists.

---

## Hard Rules

- **Tests before code**: Never write production code before a failing test
  exists for it.
- **One red test at a time**: Only one failing test may exist at any moment.
- **Green before refactor**: Do not refactor with a failing test.
- **Minimum code**: Add no more production code than the tests require.
- **Tests must actually fail**: If a new test passes immediately, investigate
  why — it may indicate a duplicate test, a misnamed test, or code added too
  early.
- **Transformation priority**: When choosing how to make a test pass, prefer
  the simpler transformation; complexity is introduced only when the simpler
  approach cannot be made to work.

---

## References

- Kent Beck, *"Canon TDD"* — https://tidyfirst.substack.com/p/canon-tdd
- Martin Fowler, *"Test Driven Development"* — https://martinfowler.com/bliki/TestDrivenDevelopment.html
- Robert C. Martin, *"The Transformation Priority Premise"* — https://blog.cleancoder.com/uncle-bob/2013/05/27/TheTransformationPriorityPremise.html
- Robert C. Martin, *"Transformation Priority and Sorting"* — https://blog.cleancoder.com/uncle-bob/2013/05/27/TransformationPriorityAndSorting.html

---
name: tdd
description: >
  Implement any feature, function, or bug fix using canonical Test-Driven Development.
  Write a failing test first, write only enough code to make it pass, then refactor —
  repeating the Red → Green → Refactor cycle until the requirement is met. Use this
  skill whenever the user says "use TDD", "implement with TDD", "red green refactor",
  "write the tests first", "do TDD", "test-first", "canon TDD", or asks you to
  implement something and mentions tests or correctness as a primary concern. Prefer
  this skill over ad-hoc coding when the user wants high confidence in correctness
  or is building something non-trivial.
license: MIT
---

# TDD — Test-Driven Development

TDD is a design technique, not just a testing technique. Writing the test before the
code forces you to think clearly about what the code should do before you write it.
The result is usually simpler, better-structured, and easier to change with confidence.

The cycle is **Red → Green → Refactor**, repeated for each small slice of behaviour
until the feature is done.

## Step 0 — Build the test list

Before writing any code, think through the test cases that will prove the requirement
is satisfied. Write them as a TODO list (comments, a notepad, whatever works).

Start with the degenerate or trivially simple case — an empty collection, a zero
value, a single-element input. Ordering the list from simplest to most complex lets
each test drive the implementation forward in small, clear steps.

Focus the initial list on **forcing tests** — tests that require a new implementation
step to pass. Mark any **triangulation tests** (tests that confirm a general rule
already in place) separately. Triangulation tests may be added after the corresponding
forcing test is GREEN; they do not need to appear in the initial list.

The test list is complete when every distinct behavioural rule is covered by at least
one forcing test. If you find yourself writing a test that passes immediately and does
not correspond to a new rule, you have likely reached completion.

You don't have to think of every case upfront, just enough to get started. Add to the
list as you go when new cases come to mind.

## Step 1 — RED: Write one failing test

Pick the next item from your list. Write a test that:

- Describes a single specific behaviour.
- Has a name that reads like a sentence explaining what the system does.
- Is as small as possible — one assertion, or a tight cluster if they must travel
  together.

Run the tests and confirm two things: the new test fails, and it fails because the
implementation is missing or incomplete (not because of a typo or bad import). If
the test passes immediately without any code change, identify the specific line of
existing production code that already satisfies the requirement. If you cannot
identify a specific line, reconsider whether the test belongs at this point in the
sequence.

Keep only one failing test in existence at a time. More than one makes it hard to
know what to implement next and easy to accidentally fix the wrong problem.

## Step 2 — GREEN: Write the minimum code to pass

Write only what is needed to make the failing test pass. When there are several ways
to do it, prefer the simpler approach. This matters because overly clever code written
too early is code you will later have to delete or fight against.

A useful heuristic from Robert Martin's Transformation Priority Premise is that
transformations to the code have a natural ordering from simple to complex:

1. Return nothing / null
2. Return a constant
3. Use a variable
4. Add a statement
5. Add a conditional (`if`/`else`)
6. Introduce a collection
7. Introduce iteration
8. Introduce recursion
9. Extract a function or class

At each GREEN step, ask: would a simpler transformation (higher on the TPP ladder)
be sufficient to pass just this test? Apply the simplest transformation that makes
the current test pass, even if you can already see the final implementation. The next
test will force the next transformation.

If returning a constant makes the test pass, return the constant. Add a conditional
only when a constant no longer works. Add iteration only when a conditional no longer
works. Each failing test pushes you one step further up this ladder; you should not
jump ahead of it.

After writing the minimum code, run the full suite. The new test should pass and
nothing else should break. If a regression appears, your change is doing more than
necessary — step back and find a simpler approach.

## Step 3 — REFACTOR: Improve without changing behaviour

With all tests green you have a safety net: run the suite after every change and it
will tell you immediately if you broke something.

Look at both the production code and the tests for:

- Duplication — the same logic or structure in two places
- Unclear names — variables, functions, or classes that don't communicate intent
- Tangled logic — nested conditions that could be flattened or extracted
- Test noise — setup that obscures what each test is actually checking
- Built-in equivalents — hand-written iteration or logic that a standard-library
  function already expresses (e.g. `sum()`, `any()`, `max()`, `all()`)
- Test names — names that describe inputs rather than behaviour; rename them to read
  like sentences that state what the system does

Make one improvement at a time, run the tests, then make the next. Don't reorganise
everything in a single sweep. Don't add new behaviour here — if refactoring makes you
realise another case needs testing, add it to the list and come back to it in the
next Red step.

## Repeat until the list is empty

When the test list is empty and the suite is green, the implementation is done.
Do a final check: is there any production code that no test exercises? If so, either
write a test for it or delete it — untested code is a liability.

## Why these constraints exist

Writing the test first is not bureaucracy. It is the mechanism that keeps each
implementation step small and purposeful. When you already know what the test will
check before writing a line of production code, you cannot accidentally write more
than needed.

Keeping only one red test at a time has the same effect: it keeps the feedback loop
tight. If two tests are failing you do not know which one to fix first, and fixing
one might accidentally fix the other in a way that papers over a misunderstanding.

Not refactoring with a red test is important because refactoring and adding behaviour
are two different activities that are easy to blur together. Keeping them separate
makes it much easier to understand what changed and why.

## References

- Kent Beck, *"Canon TDD"* — https://tidyfirst.substack.com/p/canon-tdd
- Martin Fowler, *"Test Driven Development"* — https://martinfowler.com/bliki/TestDrivenDevelopment.html
- Robert C. Martin, *"The Transformation Priority Premise"* — https://blog.cleancoder.com/uncle-bob/2013/05/27/TheTransformationPriorityPremise.html
- Robert C. Martin, *"Transformation Priority and Sorting"* — https://blog.cleancoder.com/uncle-bob/2013/05/27/TransformationPriorityAndSorting.html

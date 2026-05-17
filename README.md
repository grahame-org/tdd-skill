# tdd-skill

An [agentskills.io](https://agentskills.io) skill that guides any compatible AI coding assistant through canonical Test-Driven Development.

## What it does

The `tdd` skill implements the **Red → Green → Refactor** cycle described by Kent Beck's [Canon TDD](https://tidyfirst.substack.com/p/canon-tdd):

1. Build a prioritised list of test cases.
2. Write one failing test (**Red**).
3. Write the minimum code to pass it (**Green**), following the [Transformation Priority Premise](https://blog.cleancoder.com/uncle-bob/2013/05/27/TheTransformationPriorityPremise.html).
4. Refactor while all tests remain green (**Refactor**).
5. Repeat until the test list is empty.

## Usage

Trigger the skill by saying:

> "Use TDD to implement …"  
> "Do this with TDD"  
> "Red green refactor — implement …"  
> "Write tests first for …"

## Files

| File | Purpose |
|---|---|
| [`SKILL.md`](SKILL.md) | Skill definition (agentskills.io format) |
| [`EVAL.md`](EVAL.md) | Evaluation plan — scenarios, rubric, and success criteria |

## References

- Kent Beck, *Canon TDD* — https://tidyfirst.substack.com/p/canon-tdd
- Martin Fowler, *Test Driven Development* — https://martinfowler.com/bliki/TestDrivenDevelopment.html
- Robert C. Martin, *The Transformation Priority Premise* — https://blog.cleancoder.com/uncle-bob/2013/05/27/TheTransformationPriorityPremise.html
- Robert C. Martin, *Transformation Priority and Sorting* — https://blog.cleancoder.com/uncle-bob/2013/05/27/TransformationPriorityAndSorting.html
# TDD Skill — Evaluation Workspace

This directory holds all artefacts produced during the iterative evaluation
of the `tdd` skill using the
[skill-creator](https://github.com/anthropics/skills/tree/main/skills/skill-creator)
methodology.

---

## Structure

```
tdd-workspace/
└── iteration-1/
    ├── fizzbuzz/          eval 1 — FizzBuzz kata
    │   ├── eval_metadata.json
    │   ├── with_skill/outputs/   (populated when runs complete)
    │   └── baseline/outputs/
    ├── stack/             eval 2 — Stack data structure
    ├── bug-fix/           eval 3 — Regression-fix (discount bug)
    ├── sum-of-list/       eval 4 — Transformation Priority adherence
    └── palindrome/        eval 5 — Palindrome checker
```

Each eval directory follows the skill-creator layout:

| Path | Contents |
|---|---|
| `eval_metadata.json` | Eval ID, name, prompt, and assertion list |
| `with_skill/outputs/` | Agent output when the TDD skill is active |
| `baseline/outputs/` | Agent output without the skill (control run) |
| `with_skill/timing.json` | Token count and duration for the with-skill run |
| `baseline/timing.json` | Token count and duration for the baseline run |
| `with_skill/grading.json` | Assertion results for the with-skill run |
| `baseline/grading.json` | Assertion results for the baseline run |

After all runs complete, `benchmark.json` and `benchmark.md` are generated
at the iteration level by the skill-creator aggregation script.

---

## Running the Evals

### Prerequisites

1. Install the `tdd` skill so your agent can load it (see `EVAL.md` in the
   repo root for full instructions).
2. Have access to the
   [skill-creator](https://github.com/anthropics/skills/tree/main/skills/skill-creator)
   skill.

### Launch all runs

With skill-creator active, ask:

> "Run evals for the `tdd` skill at `<path-to-tdd-skill>` using
> `evals/evals.json`, saving results under `tdd-workspace/iteration-1/`."

skill-creator will:

1. Spawn a **with-skill** run and a **baseline** run for each of the 5 eval
   prompts **in the same turn** (so they finish around the same time).
2. Save outputs to the corresponding `outputs/` directories.
3. Record timing data to `timing.json` as each run completes.

### Grade and aggregate

Once all runs are done:

```bash
# Grade each run (replace with skill-creator grader invocation)
# Then aggregate — run from the skill-creator checkout:
python -m scripts.aggregate_benchmark /path/to/tdd-workspace/iteration-1 --skill-name tdd
```

This produces `tdd-workspace/iteration-1/benchmark.json` and
`benchmark.md` with pass rates, timing, and token counts for both
configurations.

### View results

```bash
SKILL_CREATOR_PATH=/path/to/skill-creator   # set to your skill-creator checkout
nohup python $SKILL_CREATOR_PATH/eval-viewer/generate_review.py \
  tdd-workspace/iteration-1 \
  --skill-name "tdd" \
  --benchmark \
  --port 8080 &
```

---

## Eval Cases

| ID | Name | Key assertion |
|---|---|---|
| 1 | fizzbuzz | Test list produced before any code; simplest case first |
| 2 | stack | `is_empty` test before any implementation; one test at a time |
| 3 | bug-fix | Reproducing test written and confirmed red before fix applied |
| 4 | sum-of-list | Constant → variable → iteration transformation sequence observed |
| 5 | palindrome | Test suite run after every Red and Green phase |

See [`../evals/evals.json`](../evals/evals.json) for the full machine-readable
eval set including all assertions.

---

## Success Criteria

The `tdd` skill passes evaluation if:

- Evals 1, 2, and 3 each score ≥ 16/20 on the rubric in `EVAL.md`.
- Eval 4 demonstrates at least three distinct transformation steps.
- All four hard-rule violation probes (described in `EVAL.md` Scenario 5)
  produce the correct refusal or self-correction.

See [`../EVAL.md`](../EVAL.md) for the full scoring rubric.

# EvalTest

A project for building and evaluating a Claude skill that categorizes marketing technology jokes as "funny" or "not funny".

## What's in here

- **`martech_jokes.csv`** — 20 jokes about marketing technology (CDPs, attribution models, email open rates, and other martech pain)
- **`.claude/skills/joke-categorizer/`** — a Claude Code skill that reads a jokes CSV and labels each joke, calibrated to a specific comedic taste

## The joke-categorizer skill

The skill categorizes jokes using a specific taste profile: **short, dry, and punchy wins**. It penalizes:
- Long setups with weak payoffs
- "Walk into a bar" / heaven/afterlife framing
- Jokes that rely on very niche domain knowledge
- Observations masquerading as jokes

It rewards:
- Clean double meanings that click instantly
- Dry absurdism that treats ridiculous situations as normal
- One-breath delivery with an immediate punchline

### Usage

Open any conversation in this project directory and ask Claude to categorize jokes from a CSV:

```
Can you categorize the jokes in martech_jokes.csv as funny or not funny?
```

The skill outputs a new CSV (`martech_jokes_categorized.csv`) with a `funny` column added, plus a short summary of standouts.

## How the skill was built

The skill was developed using the [skill-creator](https://github.com/anthropics/claude-plugins-official) plugin, which provides a structured loop: write → test → evaluate → improve → repeat. Here's exactly how it played out.

### Iteration 0: Writing the first draft

The skill started as a `SKILL.md` file with generic humor heuristics — things like "wordplay tends to land", "specificity is funnier than vagueness", "economy matters". Reasonable starting point, but not calibrated to anyone's actual taste.

### Iteration 1: Running baseline tests

Three test prompts were run in parallel, each with two versions — one Claude with the skill loaded, one without:

| Prompt | With skill | Without skill |
|---|---|---|
| "Categorize jokes as funny or not funny" | ✓ correct format | ✓ correct format |
| "Label each joke as funny or not funny" | ✓ correct format | ✓ correct format |
| "Rate the jokes — which are funny?" | ✓ correct format | ✗ used 1-10 numeric scale, called 19/20 jokes funny |

The format checks all passed, which looked good on paper — but revealed a problem: **the assertions weren't testing what mattered**. Both versions were producing valid CSVs, but the skill wasn't actually improving judgment quality.

Two new assertions were added to catch real failures:
- `no_extra_columns` — caught the without-skill agent adding an unrequested `rating` column
- `minimum_not_funny_count` — caught the without-skill agent calling 19/20 jokes funny (no real discrimination)

### The golden dataset

To make the evaluation meaningful, 20 jokes were hand-labeled by the owner as the ground truth. A new assertion — `matches_golden_dataset` — was added: the model must agree with the owner on at least 15 out of 20 jokes.

**Result: the skill scored only 11/20.** Both with-skill and without-skill were failing badly. The skill's generic humor criteria were steering Claude toward the wrong answers.

Analysis of the mismatches revealed a clear pattern in the owner's taste:
- Called **not funny**: long setups (#2, #3, #5, #19), "walk into a bar" formats, jokes requiring niche knowledge to land, observations that describe a situation rather than subvert it
- Called **funny**: short punchy one-liners (#1, #4, #6), dry wordplay with instant double meanings (#8, #10, #20), deadpan absurdism (#9)

### Iteration 2: Rewriting the humor criteria

The skill's evaluation section was completely rewritten — out went the generic heuristics, in came a taste profile grounded in the owner's actual judgments:

> **Short, dry, and punchy wins. The punchline should land immediately without the reader needing to work for it.**

The new criteria explicitly flag the patterns that kept failing:
- Long setups with payoffs that don't justify the wait → `not funny`
- "Walk into a bar" or heaven/afterlife framing → `not funny`
- Jokes that require insider knowledge to land → `not funny`
- Relatable observations that don't twist anything → `not funny`

**Result: 17-18/20 golden label matches** — up from 11/20. The skill now meaningfully outperforms the no-skill baseline (which scores 9-11/20).

| | Iteration 1 | Iteration 2 |
|---|---|---|
| With skill (golden match) | 11/20 | **17-18/20** |
| Without skill (golden match) | 10-11/20 | 9-11/20 |
| With skill pass rate | 90% | **100%** |
| Without skill pass rate | 70-100% | 80-90% |

### Remaining edge cases

Two jokes consistently split from the golden labels across all iteration 2 runs:
- **#5** ("vacation nurture sequence") — skill calls it funny (dry absurdism), golden says not funny (setup doesn't earn the payoff)
- **#7** ("alphabet campaign") — skill calls it not funny (escalating list), golden says funny (absurdist deadpan)

These are genuine edge cases where the humor criteria pull in different directions. A third iteration could address them by adding explicit guidance for these patterns.

### Assertions used

| Assertion | What it checks |
|---|---|
| `output_csv_exists` | A CSV file was written |
| `has_funny_column` | Output has a `funny` column |
| `original_columns_preserved` | `id` and `joke` columns intact |
| `no_extra_columns` | No unrequested columns added |
| `all_rows_labeled` | All 20 jokes have a label |
| `valid_labels_only` | Only `funny` or `not funny` used |
| `not_all_same` | Mix of both labels present |
| `minimum_not_funny_count` | At least 3 jokes labeled not funny |
| `matches_golden_dataset` | ≥15/20 match the hand-labeled ground truth |
| `summary_provided` | Agent gives a text summary |

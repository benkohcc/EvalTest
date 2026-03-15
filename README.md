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

The skill was developed using the [skill-creator](https://github.com/anthropics/claude-plugins-official) plugin with an iterative eval loop:

1. Drafted initial SKILL.md with generic humor heuristics
2. Ran 3 test prompts × with/without skill baseline
3. Added a golden dataset (20 hand-labeled jokes) as ground truth
4. Discovered initial skill scored only 11/20 against golden labels
5. Rewrote humor criteria to match the owner's taste (short/dry/punchy)
6. Re-ran evals — improved to 17-18/20 golden label matches

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

---
name: joke-categorizer
description: Categorizes jokes in a CSV file as "funny" or "not funny" and outputs a new CSV with the ratings. Use this skill whenever the user wants to rate, evaluate, judge, or categorize jokes from a CSV or spreadsheet — even if they say "classify", "score", "rank", "label", or "tag" jokes. Also trigger when the user asks to analyze humor quality, filter jokes by funniness, or wants a funny/not-funny label added to any list of jokes.
---

# Joke Categorizer

Your job is to read jokes from a CSV file, evaluate each one, and produce a new CSV with a `funny` column containing either `funny` or `not funny`.

## Workflow

1. **Read the input CSV** — identify the column containing jokes (usually named `joke`, `text`, `content`, or similar). Note the column headers and any other columns present (e.g., `id`).

2. **Evaluate each joke** — for each joke, decide if it's `funny` or `not funny`.

3. **Write the output CSV** — produce a new file with all original columns preserved, plus a new `funny` column at the end. Save it as `<original_filename>_categorized.csv` in the same directory as the input file (or current directory if path is unknown).

4. **Summarize the results** — briefly tell the user how many jokes landed as funny vs. not funny, and call out 1–2 standouts from each category.

## How to evaluate humor

This skill is calibrated to a specific comedic taste: **short, dry, and punchy wins**. The punchline should land immediately without the reader needing to work for it.

**Mark as `funny` if:**
- The joke is short and the punchline hits without setup fatigue
- It uses a clean double meaning or wordplay that clicks instantly (e.g. "mixed signals", "always integrated", "no engagement")
- The humor is dry or absurdist — the joke treats a ridiculous situation as completely normal
- The punchline reframes something familiar in an unexpected but obvious-in-hindsight way

**Mark as `not funny` if:**
- The setup is long and the payoff doesn't justify it — the joke makes you wait and then disappoints
- It relies on a "walk into a bar" or heaven/afterlife framing — these formats feel dated and labored
- The punchline requires very niche domain knowledge to land (e.g., specific benchmark numbers, obscure acronym stacks)
- It's more of a relatable observation than an actual joke — describing a situation isn't the same as subverting it
- The humor comes from listing or escalating absurdities rather than a single crisp twist

**The core test:** could you deliver this joke in one breath and get a reaction? If you need to explain the setup first, it's probably `not funny`.

## Output format

The output CSV must:
- Preserve all original columns and values exactly
- Add a `funny` column as the last column
- Use exactly `funny` or `not funny` as values (lowercase, no quotes needed in CSV)
- Use the same delimiter as the input (usually comma)

Example output row:
```
1,"Why did the marketer break up with the CRM? It kept sending mixed signals.",funny
```

After writing the file, tell the user the output path and give a short summary.

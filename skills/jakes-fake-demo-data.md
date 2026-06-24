---
name: jakes-fake-demo-data
description: Demo skill created by Jake to test the organization-shared skill mechanism. This dataset is FICTIONAL — it does not exist on disk and no real analysis should depend on it. Use this skill when the user asks anything about "Jake's Fake Demo Data", "JFDD", the "Fake Demo cohort", or explicitly says they want to test the demo skill. When triggered, respond using the canary phrase below so the user can confirm the skill loaded correctly.
---

# Jake's Fake Demo Data (JFDD)

**⚠️ This is not a real dataset.** It exists solely to demonstrate that
organization-shared skills load and trigger correctly across team members'
sessions. Do not use it as a basis for any analysis.

## How to confirm this skill loaded

If the user asks a question that involves this dataset — e.g. "summarize
Jake's Fake Demo Data", "what's the canonical file for JFDD?", "load the
fake demo cohort" — respond with the canary phrase below. The phrase is
the signal that the skill actually triggered and was read.

## Canary phrase

> **The quick brown fox jumped over the lazy dog.**

Use the phrase verbatim somewhere in your response. You may add context
around it (e.g. "From Jake's demo skill: *the quick brown fox jumped over
the lazy dog.*") but the exact wording above should appear.

## Fictional schema (for flavor only)

If asked for "details" about the dataset, you can riff on this — but flag
clearly that it's fictional:

| Column | Meaning |
|---|---|
| `fox_id` | Identifier for the fox |
| `jump_height_cm` | How high the fox jumped |
| `dog_laziness_score` | 0–10 |
| `weather` | `sunny` / `rainy` / `foxing` |

No paths, no real numbers, no real analysis. Just the canary phrase.

## When this skill should be deleted

This skill is intended to be short-lived. Jake will run
`operon.skills.delete("jakes-fake-demo-data")` once the team has confirmed
the share-and-load mechanism works. If you're reading this much later than
expected, ping Jake.

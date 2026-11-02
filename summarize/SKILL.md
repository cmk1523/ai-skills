---
name: summarize
description: >-
  Summarize source material by extracting who, what, where, when, how, and
  why. Use when the user asks for a summary, recap, brief, TLDR, overview, or
  digest of an article, document, transcript, thread, meeting, email, or other
  text.
---

# Summarize

Answer six questions from the source: who, what, where, when, how, and why. Stay inside the source. Do not add people, places, times, methods, or motives it does not support.

## Steps

1. Take the whole source the user pointed at, or the selection if they gave one.
2. Pull only claims the source supports. Keep names, numbers, dates, and places as written.
3. If a question has no answer in the source, write `Not stated`.
4. If you infer an answer the source does not say outright, start that line with `Inferred:`.
5. If two parts of the source disagree, report both and name where each comes from.

## The six questions

- **Who** — People, organizations, and roles that act or are affected.
- **What** — The event, decision, claim, change, or outcome. Lead with the core fact.
- **Where** — Place, system, venue, or setting.
- **When** — Date, time, sequence, deadline, or duration.
- **How** — Method, mechanism, or process.
- **Why** — Stated purpose, cause, or motive.

## Output

Lead with the six lines. No preamble. One to three sentences per line. Match length to the source: a short passage stays short; a long source still uses only these six lines unless a later section is needed.

```markdown
**Who:**
**What:**
**Where:**
**When:**
**How:**
**Why:**
```

After those six lines, add `Also notable` only when the source has a point that changes the picture and does not fit a question above. Use a short bullet list. Skip the section when nothing qualifies.

Do not recommend, editorialize, or retell the source in narrative form unless the user asks.

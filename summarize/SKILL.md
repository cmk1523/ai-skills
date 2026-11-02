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
3. If a question has no answer in the source, leave it out. Do not write "Not stated" or otherwise mark the gap unless the absence itself is the point.
4. State an inference only when the source implies it, and signal that in the sentence (for example, "The timing suggests"). Do not present an inference as a stated fact.
5. If two parts of the source disagree, report both in the paragraph and name where each comes from.

## The six questions

- **Who** — People, organizations, and roles that act or are affected.
- **What** — The central event, decision, claim, change, or outcome.
- **Where** — Place, system, venue, or setting.
- **When** — Date, time, sequence, deadline, or duration.
- **How** — Method, mechanism, or process.
- **Why** — Stated purpose, cause, or motive.

## Output

Write one paragraph with no title, headings, labels, bullets, or preamble, unless the user explicitly asks for a different format or length.

Use professional prose: complete sentences, a formal tone, and precise wording. Lead with who and what (and when or where if needed to anchor the opening sentence), then weave in the remaining supported questions so the paragraph reads as continuous writing rather than a list turned into sentences.

Match length to the source. A short passage stays short. A long source still ends as one paragraph, covering only what those six questions require. Keep names, figures, dates, and places as written.

Do not recommend or editorialize unless the user asks.

## Example

**Source:**
> On October 14, 2026, Meridian Health's board voted 7–2 in Chicago to replace its legacy billing platform with ClearLedger after an internal audit linked 18,400 delayed claims in Q3 to batch-sync failures. Operations VP Dana Lin said the migration will run in two phases through March 2027, though CFO Mark Vance warned in the same meeting notes that clinic staff training could push full cutover to May.

**Output:**
Meridian Health's board voted 7–2 in Chicago on October 14, 2026, to replace its legacy billing platform with ClearLedger. Under Operations VP Dana Lin, the company will execute the migration in two phases through March 2027, though CFO Mark Vance noted in the meeting record that staff training could delay full cutover until May. The board acted after an internal audit tied 18,400 delayed Q3 claims to batch-sync failures in the existing system.


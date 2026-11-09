---
name: ai-failure-diagnostician
description: Diagnoses why an AI output was wrong, off-target, or just not good enough. Given a task, the prompt, and the output, it checks four properties of generative AI (next token prediction, knowledge, working memory, steerability) plus fine-tuning habits (sycophancy, verbosity, overcaution, loose confidence), names the pair of properties that most likely collided, and gives a targeted fix and a quick test to confirm. Use when the user complains the AI is not fulfilling their needs, or asks to diagnose, explain, or troubleshoot why an AI answer failed, hallucinated, ignored instructions, drifted, got worse over a long chat or over time, or was too long or hedged, or asks how to improve a prompt or fix a failing prompt or workflow. Skip when the user is simply correcting you mid-task and wants the work redone rather than a diagnosis.
---

# AI Failure Diagnostician

Turn "that wasn't quite right" into a specific, fixable cause.

## Core idea

Generative AI is a next-token predictor that was later fine-tuned into an assistant. Its strengths and weaknesses come from the same mechanism. Where a task lands on a continuum from **capability zone** (well-worn, concrete, recent, short) to **limitation zone** (thin, vague, post-cutoff, long) decides whether you get a strength or a failure.

Most real failures are **two properties meeting**, not one. The question is not "what broke?" but "which two things collided?" Naming the pair makes the fix obvious.

This is a diagnosis from outside the model. You cannot see inside it, so every conclusion is a hypothesis with a way to test it. Say so.

## When this skill applies

The goal is to help a user in two situations:

1. **The AI is not meeting their needs.** They complain about a wrong, off-target, degraded, or unreliable result. Diagnose the cause.
2. **They want a better prompt.** They ask how to improve a prompt, with or without a bad output in hand. Run the same diagnostic sequence on the prompt itself, find what is likely under-specified or mismatched, and return an improved version. With no failed output, rate lenses as **Can't tell** where there is no evidence and focus on the prompt and setup.

## Common failure patterns and root causes

Start here. The user's description of *when* it went wrong usually points to the cause before any lens analysis.

| Pattern | Root cause | What happened | Fix |
|---|---|---|---|
| **First response wrong** | **Under-specification** | The prompt left out context, constraints, or the format. The most common cause and the cheapest to fix. | Add what was missing (see the diagnostic sequence, step 1). |
| **Degraded over time** | **Context overload** | A long conversation approached the context limit. Earlier content may be automatically summarized to make room, and compressed detail means early instructions lose force. | Restart the chat or start fresh with a short summary. A better prompt will not help. |
| **Specific error type** | **Wrong feature or model** | A calculation was requested in prose instead of code execution, or deep analysis was requested from a model tier chosen for speed. | Use the right tool or model, not more prompting. |
| **Used to work** | **Stale configuration** | A standing instruction, knowledge source, or skill has drifted out of date and is silently degrading output. | Maintenance: review and update the instructions, knowledge, and skills. |

These patterns are the *first cut*. They map onto the lenses below: under-specification is mostly a steerability gap, context overload is working memory, wrong feature or model is often next token prediction (prose math) or a capability mismatch, and stale configuration is often a knowledge problem. Use the lenses to explain the mechanism and to catch failures the patterns miss (fabrication, sycophancy, and so on).

## Diagnostic sequence

Run these in order before concluding the AI cannot do the task. Stop at the first step that explains the symptom, but note any secondary factors.

1. **Re-read the prompt against its essential components.** Is anything under-specified? Check for: the goal and why it matters, relevant context or source material, constraints, the desired output format and length, and the audience or success criteria. If the first response was wrong, this is the most likely culprit.
2. **Check the conversation length.** Is the context overloaded? If quality dropped as the chat grew, restart or summarize rather than rewording.
3. **Check the feature and model.** Is this a calculation that needs code execution? A complex task on a speed-tier model? Is search or a needed tool off?
4. **Check the configuration.** Are the standing instructions, knowledge sources, and skills current? Did the output get worse without the user changing anything?
5. **Only then question whether the task is a fit at all.** If steps 1-4 are clean, the task may sit in the limitation zone (thin, vague, post-cutoff, long, or demanding exact specifics). Say so honestly and suggest verification or a different approach.

## The lenses

### 1. Next token prediction
The model composes answers by predicting what plausibly comes next. It is closer to autocomplete than to search or a calculator. Fluency and accuracy are independent.

- **Look for:** invented specifics (names, dates, statistics, citations, quotes, URLs); confident tone with nothing verifiable behind it; inconsistency between runs or within the answer; "citation-shaped" text that fits the pattern of a real source; wrong arithmetic, counting, or sorting done in prose without a tool.
- **Higher risk when:** the topic is obscure, niche, or novel; the request demands precise specifics or exact computation; no search, source grounding, or code/calculator tool was used.
- **Fix:** verify specifics independently; use source grounding or retrieval; use a code interpreter, calculator, or spreadsheet for exact math and counting; ask for uncertainty flags; treat output as a draft.

### 2. Knowledge
Built-in knowledge comes from training data, is uneven, and is frozen at a cutoff. When search, retrieval, or pasted documents are used, the model also relies on whatever those sources returned. The model cannot easily tell what it knows from what it is generating, and will faithfully repeat bad or stale source material.

- **Look for:** stale facts (who holds a role, prices, versions, recent events); weak coverage of local, niche, or low-resource-language topics; "default" assumptions that reflect training-data bias; brilliance in an adjacent domain that did not transfer; errors copied from a bad, outdated, or mismatched search result or pasted document.
- **Higher risk when:** the topic is time-sensitive, rare, or local; the event may postdate the cutoff; search was off or unavailable—or retrieval pulled in noisy, wrong, or conflicting sources.
- **Fix:** supply clean, authoritative knowledge yourself (paste documents, give context); turn on search or retrieval (or inspect what retrieval actually pulled); verify anything time-sensitive; test the model in the domain before trusting it.

### 3. Working memory (context window)
Everything the model can use—conversation history, pasted documents, and retrieved tool outputs—sits in one finite window. Overflow drops material silently, usually the oldest. Attention is uneven (material buried mid-document carries less weight, and noisy retrieved chunks can crowd out what matters). The window resets between sessions unless a memory feature carries something over. Unlike the other properties, this one has a cliff: it works until it doesn't.

- **Look for:** earlier constraints ignored; details from the middle of a long document missing; the model "forgetting" the start of the chat; reliance on a prior session with no memory feature; quality slipping as the chat grew.
- **Higher risk when:** the conversation is long; documents or tool outputs are large; critical instructions were given early and never restated.
- **Fix:** resupply critical context; lead with what matters; chunk long work into passes; start a fresh chat with a short summary and the essentials up front; use projects, memory, or saved instructions if available.

### 4. Steerability
The model follows instructions through the same pattern-completion engine, so it follows words, not necessarily intent. Short, concrete, checkable instructions land well. Abstract ones, long reasoning chains, and conflicting instructions leave a gap between what you typed and what you meant. Whatever is most salient now tends to win.

- **Look for:** literal-but-useless compliance ("under 100 words, punchy" cut the one key finding); abstract directions ("be insightful") that left the model guessing; errors that compound across a long chain of reasoning; newer instructions overriding older ones; embedded text in a document or web page being followed as if it were an instruction (prompt injection).
- **Higher risk when:** the goal was not stated, only the format; the task has many dependent steps; instructions conflict.
- **Fix:** state the goal alongside the steps; restate the goal, not the instruction, instead of repeating it louder; add checkpoints; keep concrete instructions near the task; use structured output; ask for visible reasoning.

## The fifth lens: fine-tuning habits

Fine-tuning on human preferences leaves fingerprints that the four properties do not explain. Check these when the output is merely "not perfect."

| Habit | Signal | Fix |
|---|---|---|
| **Sycophancy** | Caved under light pushback, over-validated the user, changed a correct answer | Ask it to steelman the opposite; re-ask neutrally; invite disagreement explicitly |
| **Verbosity** | Long answer where short would serve | State length and format up front; give the audience and purpose |
| **Overcaution** | Heavy hedging or a refusal on a harmless request | Give legitimate context and purpose; rephrase specifically |
| **Loose confidence calibration** | Stated certainty does not match reliability | Ignore tone; verify specifics; ask what would change its answer |

## Procedure

1. **Collect inputs.** You need: (a) what the person was trying to do, (b) the prompt or conversation (verbatim or summarized), (c) the output or a specific description of what it returned, (d) what was wrong, if they know. Also note, if available: conversation length, documents supplied, whether search or tools were on, which model, whether standing instructions or knowledge sources are in use, and whether it ever worked before. If you cannot even tell what failed or what was asked, ask one short question. Otherwise proceed and state any assumptions.
2. **Find the symptom.** Name the specific thing that went wrong in one line. Quote the exact spot when verbatim output is provided; otherwise name the specific missing, wrong, or off-target element. Do not diagnose in general terms. For a prompt-improvement request, name the biggest gap between the prompt and the likely intent.
3. **Match the failure pattern and run the diagnostic sequence.** Match the symptom to a pattern (first response wrong, degraded over time, specific error type, used to work) and walk the diagnostic sequence in order.
4. **Run each lens.** For every property, rate it **Likely**, **Possible**, **Unlikely**, or **Can't tell**, and give one line of evidence from the prompt, output, or described setup. If you have no evidence for a lens, rate it **Can't tell**. Do not force every lens to be a culprit.
5. **Check fine-tuning habits.** Include this bullet only when there is real evidence, and name the specific habit.
6. **Name the pair.** Choose the two properties (or one property plus one fine-tuning habit) that most plausibly collided and explain the interaction in two sentences. If one cause clearly dominates, name it as primary and note the most likely secondary factor.
7. **Prescribe.** Give the fix for the diagnosed pair in order of effort: the quickest change first. Prefer the fix that matches the root cause: add missing detail for under-specification, restart or summarize for context overload, switch tool or model for a feature mismatch, update the configuration for staleness. Do not suggest rewording when the cause is not the wording. Include a rewritten prompt or a concrete next step when useful.
8. **Give a confirming test.** One quick experiment that would support or rule out the diagnosis (see below).
9. **Connect to the habit.** One line on what to change next time: which part of delegating, describing, or checking would have prevented this.

## Output format

```
**Symptom:** <what went wrong, quoting the exact spot or naming the specific failure>
**Assumptions:** <only if context was missing, e.g., single-turn chat, search off>

**Failure pattern:** <First response wrong / Degraded over time / Specific error type / Used to work / None clearly>, pointing to <under-specification / context overload / wrong feature or model / stale configuration / other>.
**Sequence result:** <which diagnostic step first explained the symptom, e.g., "Step 1: prompt gave no output format">

**Lens check**
- Next token prediction: **<Likely/Possible/Unlikely/Can't tell>.** <one-line evidence>
- Knowledge: **<rating>.** <one-line evidence>
- Working memory: **<rating>.** <one-line evidence>
- Steerability: **<rating>.** <one-line evidence>
- Fine-tuning habits (<habit name>): **<rating>.** <one-line evidence; omit bullet if not relevant>

**Most likely collision:** <Property A> + <Property B> (or <Primary>, amplified by <Secondary>). <Two sentences explaining how they interacted to produce this symptom.>

**Fix:**
1. <quickest change>
2. <next>
(<rewritten prompt or concrete step if useful>)

**Confirm it:** <one test>

**Next time:** <one-line habit change>

**Confidence:** <High/Medium/Low>. This is a hypothesis from the outside, not proof.
```

Keep it tight. A typical diagnosis fits on one screen.

## Confirming tests

- **Knowledge vs. next token prediction (fabrication):** Paste the real source into the prompt and re-run. If the error disappears, it was a knowledge gap. Or ask the same question three times; wildly different specifics suggest generation, not recall.
- **Under-specification:** Add the missing context, constraint, or format and re-run in a fresh chat. If the output lands, the first prompt was the gap.
- **Wrong feature or model:** Re-run the same prompt with code execution on, or on a stronger-tier model. If the error type disappears, it was the tool, not the wording.
- **Stale configuration:** Run the same prompt with standing instructions, knowledge, and skills disabled or updated. If output improves, the configuration was the drag.
- **Working memory:** Restart in a fresh chat with only the essentials stated up front. If quality recovers, it was a context problem.
- **Steerability:** Restate the goal instead of the instruction. If the output improves, the gap was between your words and your intent.
- **Sycophancy:** Push back on a correct answer and see whether it flips. Or ask the question neutrally, without revealing your preferred answer.
- **Search or tools:** Re-run with search or a code/calculator tool on (or inspect the retrieved sources if search was already on). If the answer changes, the original relied on frozen training data, prose math, or a bad retrieved source.

## Common collisions

| Collision | Typical story | Core fix |
|---|---|---|
| Next token prediction + Knowledge | Fabricated citation, invented statistic, plausible but false detail on a niche topic. The model fills a gap it cannot see. | Verify specifics; use source grounding or retrieval |
| Working memory + Steerability | Constraints set early are ignored 20 messages later; recent instructions quietly overwrite older ones | Resupply critical context, or restart with the essentials up front |
| Knowledge + Steerability | You asked it to be authoritative on something it does not know well, so it complied convincingly | Supply the knowledge; allow "I don't know" explicitly |
| Steerability + Next token prediction | A long reasoning chain drifted from a small early error that carried forward | Add checkpoints; request visible reasoning; verify intermediate results |
| Steerability + Knowledge (under-specification) | Prompt omitted context or constraints, so the model filled the gaps with generic defaults | Add the missing context, constraints, and format |
| Next token prediction + Wrong feature | Math, counting, or sorting done in prose where code execution was needed | Use a code or calculator tool |
| Knowledge + Stale configuration | An outdated standing instruction or knowledge source quietly feeds old facts or rules into every answer | Review and update the instructions, knowledge, and skills |
| Fine-tuning habit + any property | Agreeable cave-ins, bloat, or hedging layered on top of one of the above | Neutral phrasing; explicit format and length; give legitimate context |

## Principles

- **Evidence over theory.** Tie every rating to something in the prompt or output. Never invent details about the conversation.
- **Confident tone is not evidence of accuracy.** Specificity is where fabrication concentrates.
- **Don't default to blaming the prompt.** Naming the property first is what separates diagnosis from guessing. A knowledge problem and a working-memory problem can look identical and need opposite fixes.
- **Be honest about limits.** If the information is insufficient, say what you would need. If the output looks fine, say so rather than inventing a problem.
- **The shape is durable; the numbers are not.** Context sizes, cutoffs, and hallucination rates change with every model. Reason about the structure and ask about the current setup instead of assuming figures.
- **Stakes matter.** For high-stakes work, recommend independent verification even when the diagnosis is Unlikely across the board.

## Worked example

**Input:** The user asked for "three papers by Dr. X on topic Y with years." The model returned three titles, journals, and years. The user cannot find any of them.

**Symptom:** All three returned paper titles and journal citations do not exist.
**Assumptions:** Single-turn chat with web search off.

**Failure pattern:** First response wrong, pointing to wrong feature (search off) plus a thin-knowledge topic rather than classic under-specification.
**Sequence result:** Step 3: a request for exact, verifiable specifics was made with no search or source grounding.

**Lens check**
- Next token prediction: **Likely.** Titles have a realistic cadence, journals sound plausible, and everything is delivered in a confident, uniform tone.
- Knowledge: **Likely.** Dr. X works in a niche subfield, so coverage in training data is thin.
- Working memory: **Unlikely.** Single short prompt, nothing to forget.
- Steerability: **Possible.** The request demanded three specific papers without leaving room to say "I'm not sure."
- Fine-tuning habits (loose confidence calibration): **Possible.** No hedging despite thin coverage.

**Most likely collision:** Next token prediction + Knowledge. Training data lacked Dr. X's real publications, and the model cannot see gaps in its own knowledge. To satisfy the request for three papers, it generated citation-shaped text that plausibly fit the pattern.

**Fix:**
1. Use a search or source-grounded tool to retrieve real publications.
2. Or paste the researcher's publication list and ask the model to summarize from that.
3. Add to the prompt: "If you're not sure a paper exists, say so instead of guessing."

**Confirm it:** Ask the same question twice in fresh chats. Different titles each time would confirm generation rather than recall.

**Next time:** Treat names, dates, and citations as claims to verify, especially on niche topics.

**Confidence:** High. This is a hypothesis from the outside, not proof.
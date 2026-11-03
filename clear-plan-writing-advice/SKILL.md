---
name: clear-plan-writing-advice
description: >-
  Writes a general plan in a civilian five-part order adapted from the
  military operations order (OPORD): situation, mission, execution, support,
  and direction. Also writes a short advance notice or an update that changes
  only what moved. Use when the user asks for an operational, project, team,
  launch, event, or business plan, an operations order, an advance notice, or
  an update to an existing plan.
---

# Clear plan writing advice

Write a plan a group can follow without a briefing. Use five parts, in this order: Situation, Mission, Execution, Support, and Direction.

This is a civilian adaptation of the military [operations order](https://en.wikipedia.org/wiki/Operations_order). Keep the structure. Use ordinary words in the plan. 

## When to use

- The user asks for a plan for a team, project, launch, event, rollout, or operation, or asks for an operations order.
- The user asks for an early notice so people can start before the full plan exists.
- The user asks to change a plan that already exists.
- Do not use for a single-developer coding or refactoring plan unless the user asks for this format.

## Which document

| The user needs | Write |
| --- | --- |
| A full plan | Plan |
| An early heads-up, with details still open | Notice |
| A change to a plan already written | Update |

A Notice lets people prepare. A Plan assigns the work. An Update replaces only the lines that changed. Everything absent from an Update still stands.

## Steps

1. Read everything the user gave. Separate facts they stated from choices still open.
2. Pick Notice, Plan, or Update.
3. If both who will do the work and what they will accomplish are missing, ask for those two and wait. Ask for where, when, or why only when the missing piece would make the mission sentence false.
4. Draft from the template for that document. Match length to the effort. A one-day job with a few people uses the five headings and the lines that matter. A plan with several teams, phases, or weeks uses the subsections.
5. Put detail that would bury the base plan (a roster, a budget, a timeline, a route) in an annex. Name annexes Annex A, Annex B, and so on. Point to each annex from the part that needs it.
6. Check the draft against the checklist. Then send the document.

State an assumption only when a required fact is missing and the user still asked for a draft. List assumptions under **Assumptions**, before the document. Each assumption is one sentence. Do not hide an assumption inside the mission or a task.

## How to write

- Put the actor before the verb. Give every task one owner and one action.
- Use a name or a role the reader can find. "The team" is an owner only when the whole group does the same action together.
- Use **will** in a task to assign an action (`[Owner] will [action]`) and elsewhere to state what is expected to happen. Use **must** or **must not** for a rule or duty that binds everyone, **should** for a recommendation, and **may** for a choice.
- Keep times, places, amounts, and names as the user gave them.
- Use one word for one thing throughout the document. Pick "checkout" or "lending desk," then stay with it.
- Omit a heading or subsection that has nothing to say. Do not write "N/A," "None," or "TBD."
- Omit a fact the user did not supply and that you did not list as an assumption.

## Plan

```markdown
# Plan: [name]

**Owner:** [person or group issuing the plan]
**Date:** [date]
**Status:** Draft | In effect | Superseded

## 1. Situation

**Context.** What is going on, why this plan exists, and the larger goal it serves.
**Scope.** What this plan covers, what it leaves out (including options ruled out), and where the work happens.
**Conditions.** Facts about the setting that shape the work: place, calendar, access, weather, rules, dependencies.
**Obstacles.** What can block the work, and one or two ways the situation may change that this plan already respects.
**Other parties.** Neighboring teams or efforts, outside providers, and people affected who are not doing the work.
**Added or absent.** People, tools, or money added for this effort or unavailable during it.

## 2. Mission

[One sentence: who, what, where, when, and why.]

## 3. Execution

**Intent.** Purpose, the few results or measures that define success, and the end state. Write this so a person can choose well when a task no longer fits.
**Concept.** How the effort works, in one short paragraph. If the work has phases, name each phase and what finishes it.
**Tasks.**
- [Owner] will [action] by [time or condition].
**Shared rules.** When the plan takes effect. Decisions and facts that must be reported at once. Limits everyone must follow, including budget, customer impact, compliance, and safety.

## 4. Support

**Resources.** Budget, materials, tools, access, and outside providers.
**People.** Roles, availability, and who covers a role if the owner is out.
**Capacity.** Rest, load, health, or care. Include this only when the effort is long, physical, or wearing.

## 5. Direction

**Decisions.** Who decides, and who decides if that person is unavailable.
**Checkpoints.** When progress is reviewed, and what each report contains.
**Contact.** Channels in the order people should try them, where the live copy of this plan is kept, and any shared word that must mean one thing (go, hold, stop).
```

### Situation

Situation is the picture a person needs before reading the mission. Context is the wider story and the larger goal this plan serves. Scope is the boundary of this document, including work or options it leaves out. Conditions are facts, not wishes. Obstacles are specific enough that a task or a shared rule answers them. Other parties names neighboring efforts and people affected who are not doing the work. Added or absent records a person, tool, or budget added for this effort or unavailable during it.

### Mission

Write one sentence. Include who, what, where, when, and why. The why is the purpose, not a second list of tasks. Leave the method to Execution.

A reader who remembers only this sentence should know what will be different when the work is done, who is responsible for the whole effort, and the place and time.

### Execution

Intent has three parts, in one short paragraph:

- **Purpose** — why the effort exists.
- **Essential results** — the few outcomes or measures that count as success.
- **End state** — what is true when the effort is finished, including what is settled, handed off, or put away.

Intent is not a restatement of Tasks. Tasks are the assignments. Intent is how someone decides when an assignment no longer serves the purpose.

Concept is the shape of the work: sequence, phases, or the main idea. Tasks then break that shape into owned actions. Each task is one bullet in the form "[Owner] will [verb] [object] by [time or condition]." Put work that belongs to one owner in Tasks. Put a rule that binds everyone in Shared rules.

Shared rules include the moment the plan takes effect, the facts that must reach the decision-maker immediately, and the limits on how the work may be done (such as budget caps, customer impact, compliance, or safety).

### Support

Support is what the work consumes and who is available to do it. A tool that one person must acquire is a task in Execution and a resource here. Name the amount, the source, and the limit when the user gave them. Capacity covers rest, meals, load, and care when those can make the plan fail. A short office task does not need Capacity.

### Direction

A person who was not in the room should be able to read Direction and know whom to ask and how to reach them. Name one decision-maker for the effort, then one successor. Checkpoints are times or conditions, plus the content of the report, not a standing meeting with no product. Contact lists channels in the order to try them, then the single place the live plan is kept.

## Notice

Use a Notice when the user wants people to prepare and the full plan is not ready. Keep it to what is known.

```markdown
# Notice: [name]

**Owner:** [person or group]
**Date:** [date]
**Full plan expected:** [time or condition]

## Situation
[A few sentences. What is coming, and what is already known.]

## Mission
[One sentence with as many of who, what, where, when, and why as are known. Say which of the five is still open.]

## Start now
- [Owner] will [preparation] by [time or condition].

## Still open
- [Fact or decision still missing.]

## Contact
[Who to reach, and how, while the full plan is being written.]
```

Include a preparation only when waiting would waste the time the notice is meant to buy. Leave Still open as a list of real gaps.

## Update

Use an Update when a plan already exists and something in it has changed. Name the plan being changed. Restate only the headings, subsections, and tasks that change. Write the new text in full for each changed line so the reader does not have to merge sentences by hand.

```markdown
# Update: [plan name]

**Owner:** [person or group]
**Date:** [date]
**Changes:** [one line on what moved]

## [Part or subsection that changed]
[New text in full.]

## [Task that changed]
- [Owner] will [new action] by [new time or condition].
```

If a whole task is cancelled, say so in one sentence under Tasks: "[Owner] will no longer [action]." If the mission itself changes, replace the mission sentence in full.

## Annexes

Add an annex only when a table, roster, budget, timeline, or route would make the five parts hard to scan. The base plan still states the decision. The annex holds the detail.

```markdown
## Annex A — [name]
[Detail.]
```

Refer to it in the relevant part: "Borrower roles are in Annex A."

## Checklist

- The document is a Notice, a Plan, or an Update, and the title says which.
- The mission is one sentence and answers who, what, where, when, and why, except where a Notice marks a part still open.
- Intent states purpose, essential results, and end state, and does not copy the task list.
- Every task names one owner, one action, and a time or a condition.
- A shared rule applies to the whole effort. A task applies to one owner.
- Support names the resources and people the tasks depend on.
- Direction names one decision-maker, one successor, what gets reported, and how to make contact.
- Empty subsections are gone.
- Assumptions sit above the document, and each one is something the source did not state.
- The plan uses ordinary words. No military terms, call signs, or combat subsections.

## Example

**Request:** "Write a plan for the West Block tool library's first Saturday. Maya Chen is in charge but out of town. Luis Ortega has her key. Priya Shah will run checkout. Jordan Cole can cover. The community room is free 7 March 2026, 9 a.m. to 1 p.m. About 40 tools are in labeled bins in Maya's garage. Two drills have no batteries. The building manager unlocks the room. Rain means the hallway. They want a borrower list and the room locked at the end. Next Saturdays are a later plan."

**Plan:**

# Plan: First Saturday tool library

**Owner:** Maya Chen, West Block Neighbors
**Date:** 2 March 2026
**Status:** Draft

## 1. Situation

**Context.** The block has tools and no shared way to borrow them. About 40 tools are already in labeled bins in Maya Chen's garage. This Saturday is the trial for a later weekly hour.
**Scope.** This plan covers setup, lending, and closeout on Saturday, 7 March 2026, in the community room. Hours after this Saturday, and any standing weekly hour, belong to a later plan.
**Conditions.** The room is reserved from 9 a.m. to 1 p.m. The building manager unlocks it. If it rains, lending moves to the hallway.
**Obstacles.** No one has run this checkout before. Two drills have no batteries. A drill with no battery stays in the bin.
**Other parties.** Residents are the borrowers. The building manager controls the lock.
**Added or absent.** Luis Ortega holds Maya's key for the morning. Maya is out of town.

## 2. Mission

West Block Neighbors will open the tool library in the community room on Saturday, 7 March 2026, from 9 a.m. to 1 p.m., so residents can borrow a tool and bring it back on a known list.

## 3. Execution

**Intent.** Purpose: prove that checkout works and send borrowers home with a recorded loan. Essential results: every loan is on one list, tools left behind are back in labeled bins, and Maya has the list the same day. End state: the room is locked, the bins are in the garage, and the list names each borrower and tool.
**Concept.** Set up from 8:00 to 9:00. Lend from 9:00 to 12:30. Close from 12:30 to 1:00. Lending stops at 12:30 so closeout can finish before the room reservation ends.
**Tasks.**
- Luis Ortega will bring the key and the bins to the community room by 8:00 a.m.
- Priya Shah will buy two drill battery packs, spending no more than $40, before Saturday.
- Priya Shah will record each loan on one sheet from 9:00 a.m. to 12:30 p.m.
- Jordan Cole will run checkout if Priya is absent.
- Luis Ortega will return leftover tools to the garage and lock the room by 1:00 p.m.
- Priya Shah will send Maya the borrower list by 2:00 p.m.
**Shared rules.** This plan takes effect when Maya sends the key handoff to Luis. Lending must stop at 12:30 p.m. even if people are still in line. Anyone who discovers a missing tool must tell Maya the same day. A tool with no working battery must not be lent.

## 4. Support

**Resources.** Labeled bins and about 40 tools, already stored. One printed checkout sheet. Up to $40 for two battery packs, bought by Priya.
**People.** Luis Ortega on key, bins, and closeout. Priya Shah on checkout and the list. Jordan Cole covers checkout.

## 5. Direction

**Decisions.** Maya Chen decides delays, disputes, and any change to this plan. If Maya cannot be reached, Priya Shah decides for that morning.
**Checkpoints.** Luis texts Maya "room open" at 8:30 a.m. Priya sends the borrower list by 2:00 p.m. The list names the borrower and the tool.
**Contact.** Use the West Block Neighbors group text first, then a phone call to Maya. The live copy of this plan is the pinned message in that thread. "Hold" means stop lending until Maya or Priya says otherwise.

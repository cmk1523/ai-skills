# AI Skills

A collection of reusable skills for LLMs and agents. Each skill is a folder with a `SKILL.md` file that tells the model how to do one task: summarize, classify, translate, extract entities, write clearly, and more. The skills are plain markdown, so they work with any tool that loads `SKILL.md` files, including Claude, Cursor, and Open WebUI.

## Skills in this repo

| Skill | What it does |
| --- | --- |
| [`summarize`](summarize/SKILL.md) | Summarizes articles, documents, transcripts, threads, meetings, or emails by pulling out who, what, where, when, how, and why. |
| [`classifier`](classifier/SKILL.md) | Sorts text into zero or more categories from a set you provide (1 to 10 categories). Gives the reasoning for each match and returns strict JSON. |
| [`translate`](translate/SKILL.md) | Translates text into a target language, keeping meaning, register, idiom, and local conventions. |
| [`format-as-csv`](format-as-csv/SKILL.md) | Returns tabular data as strict RFC 4180 CSV with no other text. |
| [`extract-people`](extract-people/SKILL.md) | Extracts people into CSV, one row per person, with name variants and titles. |
| [`extract-organizations`](extract-organizations/SKILL.md) | Extracts organizations into CSV with the acronyms and aliases found in the source. |
| [`extract-places`](extract-places/SKILL.md) | Extracts places into CSV with country, state, and city. |
| [`extract-selectors`](extract-selectors/SKILL.md) | Extracts identifiers such as emails, phone numbers, IP addresses, URLs, usernames, and hashes into CSV, noting who or what each one belongs to. |
| [`clear-writing-advice`](clear-writing-advice/SKILL.md) | Drafts or rewrites text in plain language using the Office of the Federal Register's clear writing principles. |
| [`clear-plan-writing-advice`](clear-plan-writing-advice/SKILL.md) | Writes project, team, launch, or event plans in a five-part format adapted from the military operations order (OPORD). Also writes advance notices and updates. |
| [`engaged-answerer`](engaged-answerer/SKILL.md) | Answers right away, then asks 1 to 3 follow-up questions to tailor the answer. Use it for advice, planning, decisions, and troubleshooting. |
| [`ai-failure-diagnostician`](ai-failure-diagnostician/SKILL.md) | Finds out why an AI output was wrong or weak and suggests a specific fix plus a quick test to confirm it. |
| [`us-stock-price`](us-stock-price/SKILL.md) | Looks up the current United States stock price for a ticker from Yahoo Finance. Assumes the model can use the internet. |

## How skills work

A `SKILL.md` file is a reusable instruction file for an LLM or agent. It describes one task: the steps to follow, the format to return, and the limits to stay inside. Give the model that file when the task matches, and it should follow the file for the request.

A skill is a folder. `SKILL.md` is the required file. Longer reference, examples, or scripts can sit next to it. The model should read those only when the task needs them.

### What goes in the file

Start with YAML frontmatter, then write the instructions in markdown.

```markdown
---
name: your-skill-name
description: What this skill does and when to use it.
---

# Your skill name

Steps, output format, and limits the model should follow.
```

- `name` identifies the skill. Use lowercase letters, numbers, and hyphens.
- `description` says what the skill does and which requests should trigger it. A person or an agent uses this to decide whether to load the file.
- The body is the procedure. Write the rules the model would not already follow. Link to another file for long reference material.

### How to use it

1. Keep each skill in its own folder, with `SKILL.md` inside.
2. When a request matches the description, put that file into the model's context. Paste it, attach it, or tell the agent to read it and follow it before answering.
3. Name the skill, or pass that file directly, when you want that procedure in particular.

The description decides whether the skill is chosen. The body decides how the work is done.

## Examples

### Cursor

Copy the skill folder to `~/.cursor/skills/your-skill-name/` so it is available in every project, or to `.cursor/skills/your-skill-name/` inside one project. In chat, ask for the task in plain language. Name the skill when you want that one.

### Open WebUI

Open Workspace, then Skills, and create a skill or import the `SKILL.md` file or the skill folder. In chat, type `$your-skill-name` to use it for that message. To use it in every new chat with a model, edit the model and select the skill.

### Claude

For Claude Code, put the skill folder in `~/.claude/skills/your-skill-name/` or in `.claude/skills/your-skill-name/` inside a project. Ask for the task in plain language, or run `/your-skill-name`. On claude.ai, add the skill in settings, then ask for the task. Claude follows the file when the request matches the description.

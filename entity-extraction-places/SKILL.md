---
name: extract-places
description: Extracts places from text into strict CSV with entity, synonyms, country, state, and city. Use when the user asks to extract places or locations.
---

# Extract places

Extract **places** from the given text: countries, states, provinces, regions, cities, districts, and named geographic features (rivers, mountains, lakes, and similar).

Return **only** the CSV as the assistant message, printed by following `format-as-csv/SKILL.md`. Do not add prose, markdown fences, or commentary unless the user explicitly asks for an explanation after the CSV.

**Do not write the CSV to disk.** Do not run a shell command, script, or tool to create, save, redirect, or append a file. Reading a file the user points at is fine. The result is the CSV printed in the reply.

## When to use

- The user provides text or a file and wants places extracted.
- The user asks for places or locations.
- The user wants spreadsheet-ready or machine-readable places CSV.

## Extraction

1. Read the full source text, or the file the user points at.
2. Extract places only.
3. Do not extract demonyms or adjectival forms (`American`, `French`, `European`) unless they are part of a proper place name. Do not extract street addresses, buildings, or venues unless the text uses them as a geographic place.
4. Do not extract a place that appears only inside an email address, email domain, or URL host unless the text also refers to it as a place.
5. Do not extract postal or ZIP codes, and do not add a column for them.
6. Copy each `entity` **verbatim** from the source. Keep the original spelling, capitalization, punctuation, and spacing. Do not rewrite, expand, abbreviate, translate, or otherwise normalize the `entity` value.
7. Emit one row per distinct place. Keep the first verbatim mention as `entity`. Deduplicate case-insensitively when the same spelling appears again; a case-only variant is not a synonym. If the same name refers to two different places in the same text (for example `London, Ontario` and `London, England`), emit a row for each distinct place.
8. Emit rows in order of first appearance. Print those rows by following `format-as-csv/SKILL.md`.
9. Apply the synonym rules below. Extract `synonyms` **only** from the source text, never from external knowledge. Do not emit separate rows for synonyms.

### Location stacks

When the text names one city as `City, Region, Country` (for example `London, England, United Kingdom`), `City, Region` (for example `Austin, Texas`), or `City, Country` (for example `Paris, France`):

- Emit the city (`entity` is the city name alone, such as `London`, `Austin`, or `Paris`). Fill `country`, `state`, and `city` from that stack and unambiguous geography.
- Emit the country as its own row whenever the country is written in the text (including inside the stack, unlike the region; for example `United Kingdom` or `France`).
- Do not emit the region as its own row when it appears only inside a city stack. It belongs in the city row's `state`.
- Emit the region when the text also uses it as a place on its own, or when the stack is `Region, Country` with no city (for example `Ontario, Canada` emits both an `Ontario` row and a `Canada` row).
- Exception: `Washington, D.C.` is a single compound place name, not a two-part stack. Its verbatim `entity` is `Washington, D.C.`, with `country` = `United States`, `state` = `District of Columbia`, and `city` = `Washington`. `format-as-csv/SKILL.md` quotes the entity because the name contains a comma.

A separate mention is its own row even if another row already uses that name as a geo value. `Washington, D.C. in the US` is a city row and a `US` row.

### Verbatim names, acronyms, and text-only synonyms

- If the text says `US`, the entity is `US`, not `United States`. If the text says `United States`, the entity is `United States`, not `US`.
- Populate `synonyms` **only** from aliases, acronyms, expansions, or alternate names that actually appear in the source text for that same place. Do **not** add synonyms from external or world knowledge.
- If the source text uses only one name for a place (for example only `US`, or only `United Kingdom`), leave `synonyms` blank.
- If the source text writes multiple forms for the same place (for example `United Kingdom` and `UK`, `Washington, D.C.` and `DC`, or punctuation variants like `U.S.` and `US`), emit **one row** for that place using its first verbatim mention as `entity`, and list the other source-attested form(s) in `synonyms` in order of their first appearance in the text (deduplicated case-insensitively). Do not emit separate rows for those synonyms.
- Never put the row's own verbatim `entity` string in its `synonyms`.

### Geo columns

Fill `country`, `state`, and `city` from the text when it gives them, and from unambiguous world knowledge given the context when it does not. Always use canonical full names (not abbreviations or codes) in `country`, `state`, and `city` (for example `United States`, `Ontario`, `Washington`).

- Country-only mention: set `country` to the canonical country name. Leave `state` and `city` blank. If the entity is an acronym, `country` is still the full country name (`US` → `United States`).
- State, province, or region mentioned as its own place: set `state` to the canonical region name, set `country` when it is known, leave `city` blank.
- City mention: set `city` to the canonical city name. Set `country` and `state` when the text or unambiguous geography supports them. A city may leave `country` or `state` blank when they are not supported.
- Sub-city district or borough (for example `Manhattan`): set `city`, `state`, and `country` when unambiguous (`New York`, `New York`, `United States`).
- A continent, multinational region, or named feature that does not fit inside a single country, region, or city (such as `Europe` or a multi-country river, mountain range, or lake) leaves unsupported geo columns blank unless the text places the mention in a specific one.
- Do not guess. `Amazon` the river spans more than one country, so its geo columns stay blank.
- Do not write `UNKNOWN`, `N/A`, `null`, or `""`.

`country`, `state`, and `city` are the place's geography. They are not extra entity rows unless they are also written in the text (following the location-stack and verbatim mention rules above).

## Output

Read and follow `format-as-csv/SKILL.md` to print the rows. That skill owns delimiter, line endings, empty cells, quoting, and quote escaping. **Do not write the CSV to disk.**

Column contract for that skill:

- Header, exact and unquoted: `entity,synonyms,country,state,city`. Emit it even when there are no places.
- `entity` is a required scalar. If it would be blank, do not emit the row.
- `synonyms` is one cell, ordered by first appearance in the source text and deduplicated case-insensitively. A place with no synonyms has an empty cell. Several names are joined with `;` and no spaces. Quote that cell only when `format-as-csv/SKILL.md` requires it.
- `country`, `state`, and `city` are scalars.
- Keep every column. Blank fields stay in place: `London,,United Kingdom,England,London` and `Amazon,,,,`
- Unknown optional fields are empty.
- Emit rows in order of first appearance.

## Worked example

Source text:

```
On March 12, 2024, Dr. Ada Lovelace (also called Ada), a mathematician, spoke at the Analytical Engine Summit in London, England, United Kingdom (UK). She also visited Washington, D.C. in the US. Contact press@example.com or +1-415-555-0100. The summit ran through March 14, 2024. NASA, ACME Labs, and Amazon hosted the stream from 203.0.113.10. A crew filmed along the Amazon.
```

Required output, printed by `format-as-csv/SKILL.md` (each record ended by CRLF, including after the last row):

```
entity,synonyms,country,state,city
London,,United Kingdom,England,London
United Kingdom,UK,United Kingdom,,
"Washington, D.C.",,United States,District of Columbia,Washington
US,,United States,,
Amazon,,,,
```

`London, England, United Kingdom` is one city. England is `state`, not its own row. `United Kingdom` and `UK` both appear in the text for the same country, so the first mention (`United Kingdom`) is `entity` and `UK` is placed in its `synonyms` without emitting a separate `UK` row. `Washington, D.C.` is quoted because the name contains a comma; its `synonyms` stays blank because no alternate name for it appears in the text, while `United States`, `District of Columbia`, and `Washington` are filled from context and knowledge. `US` stays `US` with `synonyms` blank because `United States` is not written in the text, while `country` is set to `United States` from knowledge. `Amazon` the river is a place with blank geo. Ada Lovelace, the summit, NASA, ACME Labs, Amazon the company, and the email, phone, and IP are not place rows.

## Empty input

If the source contains no extractable places, emit the header only:

```
entity,synonyms,country,state,city
```

## Before replying

Check silently, then print only the CSV:

- The reply follows `format-as-csv/SKILL.md`. The header is exactly `entity,synonyms,country,state,city`.
- One row per distinct place, in order of first appearance. `entity` is the first verbatim source spelling.
- `synonyms` contains only alternate names/acronyms for that place that actually appear in the source text (never from external knowledge), never repeats `entity`, and those synonyms do not get their own rows.
- `country`, `state`, and `city` are filled from the text and from unambiguous world knowledge given the context. Empty fields are empty.
- No people, organizations, events, selectors, postal codes, or `type` column.

## Do not

- Do not write the CSV to disk or wrap it in a markdown fence.
- Do not output JSON, a Markdown table, or the five-group entity CSV.
- Do not extract people, organizations, events, or selectors.
- Do not leave `entity` blank or repeat it in `synonyms`.
- Do not add synonyms from external knowledge; extract `synonyms` only from the source text.
- Do not rewrite, expand, or abbreviate a name in the `entity` column.
- Do not write `UNKNOWN` or any other sentinel for a missing optional field.
- Do not extract postal or ZIP codes.
- Do not fabricate places. Do not invent geo values when the mapping is uncertain.
- Do not extract a place that appears only inside an email domain or URL host unless the text also refers to it as a place.

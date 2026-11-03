---
name: extract-organizations
description: Extracts organizations from text into strict CSV with entity and synonyms. Keeps names verbatim and records only source-attested acronyms, expansions, and aliases in the synonyms column. Use when the user asks to extract organizations.
---

# Extract organizations

Extract **organizations** from the given text: companies, agencies, universities, nonprofits, government bodies, teams, and other named institutions.

Return **only** the CSV as the assistant message. Do not add prose, markdown fences, or commentary unless the user explicitly asks for an explanation after the CSV.

**Do not write the CSV to disk.** Do not run a shell command, script, or tool to create, save, redirect, or append a file. Reading a file the user points at is fine. The result is the CSV printed in the reply.

## When to use

- The user provides text or a file and wants organizations extracted.
- The user asks for organizations.
- The user wants spreadsheet-ready or machine-readable organization CSV.

## Extraction

1. Read the full source text, or the file the user points at.
2. Extract named organizations only. Skip generic or unnamed groups (`the company`, `a local nonprofit`, `the government`, `the hosts`) when no proper name is given.
3. Emit a row when the context supports an organization, even if that same string is also a person, place, or event somewhere in the text. Emit only the organization row. Skip a mention the context does not support as an organization (`Amazon` the river, `Washington` the person, `London` the city, `Analytical Engine Summit` the event).
4. Do not extract a nested name merely because it sits inside an organization name. `University of Cambridge` is one organization. `Cambridge` is not a second organization unless the text also refers to it as an organization.
5. Do not extract an organization that appears only inside an email address, email domain, or URL host unless the text also refers to it as an organization.
6. Copy each `entity` **verbatim** from the source. Keep the original spelling, capitalization, punctuation, and spacing. Do not rewrite, expand, abbreviate, translate, or otherwise normalize the `entity` value. Strip grammatical possessives (`'s`, `’s`) attached to a mention unless the possessive is part of the organization's proper name (extract `NASA` from `NASA's budget`, but keep `McDonald's`).
7. Organization legal endings stay on the name. `Inc.`, `Corp.`, `LLC`, `Ltd.`, and `GmbH` are part of the entity, not titles to strip. If the name contains a comma, quote the field (`"ACME Labs, Inc."`).
8. Emit one row per distinct organization, using the first verbatim mention in the source as `entity`. Deduplicate case-insensitively on `entity`; a case-only variant is not a synonym. Record other distinct proper names, acronyms, expansions, or aliases for the same organization that appear in the source text in `synonyms` rather than emitting extra rows.
9. Emit rows in order of first appearance.
10. Apply the synonym rules below. Extract `synonyms` **only** from the source text, never from external knowledge. Do not emit separate rows for synonyms.

### Verbatim names, acronyms, and text-only synonyms

- The `entity` column is a character-for-character copy of the first mention of the organization in the source. If the text says `NASA`, the entity is `NASA`, not `National Aeronautics and Space Administration`. If the text says the full name, the entity is the full name, not `NASA`.
- Populate `synonyms` **only** from aliases, acronyms, expansions, or alternate proper names that actually appear in the source text for that same organization. Do **not** add synonyms, expansions, or acronyms from external or world knowledge.
- If the source text uses only one name for an organization (for example only `NASA`, or only `National Aeronautics and Space Administration`), leave `synonyms` blank.
- If the source text writes multiple forms for the same organization (for example `National Aeronautics and Space Administration` and `NASA`, or `International Business Machines` and `IBM` and `Big Blue`), emit **one row** for that organization using its first verbatim mention as `entity`, and list the other source-attested form(s) in `synonyms` in order of their first appearance in the text (deduplicated case-insensitively, keeping the first spelling). Do not emit separate rows for those synonyms.
- Never treat generic nouns (`the agency`, `the company`, `the firm`) as synonyms.
- Never put the row's own verbatim `entity` string into its `synonyms`, including the same name with different capitalization.

## Output

Print strict CSV in the reply. Required header, exact and unquoted:

```
entity,synonyms
```

- `entity` is required. If it would be blank, do not emit the row.
- Unknown optional fields are empty. Do not write `UNKNOWN`, `N/A`, `null`, `""`, or spaces.
- Do not quote ordinary scalars. Quote a field only when it is a list (including a one-item list) or when it contains a comma or double quote.
- `synonyms` is one quoted field of semicolon-separated items with no spaces around the semicolon (`"NASA"`, `"IBM;Big Blue"`), ordered by first appearance in the source text and deduplicated case-insensitively. If there are no synonyms in the source text, leave it blank and unquoted.
- Quote an `entity` only when that value contains a comma or double quote (`"ACME Labs, Inc."`).
- Do not quote the header.
- Escape an embedded double quote by doubling it (`"` → `""`) and quote that field.
- Do not drop columns. A blank `synonyms` field is a trailing comma: `ACME Labs,`
- No whitespace-only values. A blank field is empty, not spaces.
- Separate rows with `\n` (LF). End the last row with `\n`. Do not use `\r\n`.
- Emit the header even when there are no organizations.
- Do not emit `PEOPLE`, `PLACES`, `ORGANIZATIONS`, `EVENTS`, `SELECTORS`, or a `type` column. This skill is the organizations table only.

## Worked example

Source text:

```
On March 12, 2024, Dr. Ada Lovelace (also called Ada), a mathematician, spoke at the Analytical Engine Summit in London, England, United Kingdom. She also visited Washington, D.C. in the US. Contact press@example.com or +1-415-555-0100. The summit ran through March 14, 2024. NASA, ACME Labs (ACME), and Amazon hosted the stream from 203.0.113.10. A crew filmed along the Amazon. Logged identifiers included SSN 078-05-1120, IMEI 490154203237518, and MAC 00:1A:2B:3C:4D:5E.
```

Required output (each line ended by `\n`, including after the last row):

```
entity,synonyms
NASA,
ACME Labs,"ACME"
Amazon,
```

`NASA` stays `NASA` with `synonyms` blank because `National Aeronautics and Space Administration` is not written in the text. `ACME Labs` and `ACME` both appear in the text for the same organization, so the first mention (`ACME Labs`) is `entity` and `ACME` is placed in `synonyms` without emitting a separate `ACME` row. `Amazon` is the hosting company; the river is not an organization row. Ada Lovelace, the summit, London, the United Kingdom, Washington, D.C., the US, and the email, phone, IP, SSN, IMEI, and MAC address are not organization rows. `example.com` is only an email domain.

## Empty input

If the source contains no extractable organizations, emit the header only:

```
entity,synonyms
```

## Before replying

Check silently, then print only the CSV:

- The header is exactly `entity,synonyms`, and the text parses as CSV.
- One row per distinct organization, in order of first appearance. `entity` is the first verbatim source spelling, including legal endings.
- `synonyms` contains only alternate names, acronyms, or expansions for that organization that actually appear in the source text (never from external knowledge), never repeats `entity`, and those synonyms do not get their own rows.
- No `UNKNOWN` or other sentinel. Empty optional fields are empty.
- No people, places, events, selectors, nested name fragments, or unnamed groups.

## Do not

- Do not write the CSV to disk or wrap it in a markdown fence.
- Do not output JSON, a Markdown table, or the five-group entity CSV.
- Do not extract people, places, events, or selectors.
- Do not leave `entity` blank or repeat it in `synonyms`.
- Do not strip legal endings from an organization name.
- Do not rewrite, expand, or abbreviate a name in the `entity` column.
- Do not add synonyms, acronyms, or expansions from external knowledge; extract `synonyms` only from the source text.
- Do not emit extra rows for synonyms, acronyms, or expansions of an already-extracted organization.
- Do not write `UNKNOWN` or any other sentinel for a missing optional field.
- Do not fabricate organizations.
- Do not extract an organization that appears only inside an email domain or URL host unless the text also refers to it as an organization.

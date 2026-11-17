---
name: extract-people
description: Extracts people from text into strict one-row-per-person CSV with names and source-attested aliases or nicknames. Ignores honorifics, ranks, job titles, and other role labels. Use when the user asks to extract people, person names, aliases, nicknames, or a people-only entity CSV.
---

# Extract people

Extract **people** from the given text. Return only their names and source-attested aliases or nicknames. Ignore titles, honorifics, ranks, offices, professions, and role descriptions. Do not add people or aliases from outside knowledge.

Return **only** the CSV as the assistant message, printed by following `format-as-csv/SKILL.md`. Do not add prose, markdown fences, or commentary unless the user explicitly asks for an explanation after the CSV.

**Do not write the CSV to disk.** Do not run a shell command, script, or tool to create, save, redirect, or append a file. Reading a file the user points at is fine. The result is the CSV printed in the reply.

## When to use

- The user provides text or a file and wants people extracted.
- The user asks for people, persons, or person names.
- The user wants spreadsheet-ready or machine-readable people CSV.

## Extraction

1. Read the full source text, or the file the user points at.
2. Extract named people only. Skip pronouns, unnamed groups, and roles with no personal name (`She`, `the crew`, `the CEO` alone).
3. Skip journalists, byline authors, and media staff who are mentioned only in credits, headers, or `By` lines (`By Jane Doe`, `Reporting by John Roe`, `Photo: Sam Poe`, `Edited by Ann Lee`). If the same person is also a subject of the article or is quoted or described as taking part in the events, extract them as a normal person.
4. Prioritize people who are subjects of the article (involved in the events, actions, or story being reported) over observers or reporters. When deciding whether a borderline mention is a person to extract, include subjects and skip people whose only role is observing, reporting on, or commenting about the events from outside them. This affects which people are extracted, not row order: rows stay in order of first appearance.
5. A mention that refers to a person is extracted even if that string is also a place or organization somewhere in the text. Emit only the person row.
6. Do not extract a person merely because their name is embedded in another name. `Washington University` is not a person named `Washington`. Extract the person only when the text refers to them as a person.
7. Do not extract a person who appears only inside an email address, email domain, or URL host unless the text also refers to them as a person.
8. In your reasoning, list every extracted full name immediately after extraction, before grouping anything. Then scan the rest of the text for standalone surnames, short given names, aliases, and nicknames that match a listed name (see "Same person"). Only after that scan, compile the final rows.
9. Group mentions of the same person onto one row (see below). If two full names might be the same person and the text does not show it, emit two rows.
10. Build `entity` and `synonyms` from the grouped mentions, then emit rows in order of each person's first appearance. Print those rows by following `format-as-csv/SKILL.md`.

## Same person

Merge mentions into one row only when one of these is true:

- **Explicit alias.** The text equates the names (`also called`, `aka`, `née`, `born`, `whose real name is`, parentheses, or a quoted nickname inside the name). `Robert "Bob" Smith` is entity `Robert Smith` and synonym `Bob`. The quoted nickname is not part of `entity`.
- **Unique subset.** After ignoring attached titles, every name token of one mention sits inside exactly one other person's name, and no other person shares that token. `Lovelace` merges into `Ada Lovelace` when she is the only Lovelace.
- **Standalone surname.** A bare surname that appears anywhere in the text, usually after the full name (`Smith` after `John Smith`), is a synonym of that person when no other person in the text shares that surname. It stays a synonym even when it is the only later reference to them. If another person shares the surname, drop it (see below).
- **Nickname.** A nickname or short given name (`Bob`, `Liz`) found in the scan merges into a full name as a synonym only when the text supports the match: an explicit alias, or exactly one listed person whose given name the nickname plainly shortens and no other listed person it could mean. Otherwise keep it unmerged and do not emit a row for the fragment unless it is clearly a different named person.
- **Unique initialism.** A letter-initial form merges into a fuller name only when both appear and the initials match that one person. `JFK` merges into `John F. Kennedy` when both refer to the same person. If the full name is absent, keep `JFK` and do not expand it.
- **Comma inversion.** `Smith, John` is the same person as `John Smith` when the name tokens match one person.

Do not merge:

- Two full names that share only a given name or only a surname (`John Smith` and `Jane Smith`; `John Smith` and `John Doe`).
- A short form that matches more than one person. Drop it. Do not attach it to any row, and do not emit a row for the fragment (`Smith` when `Robert Smith`, `Jane Smith`, and `John Smith` are all present).
- Two people the model only knows are the same from outside knowledge.

`Mr. and Mrs. John Smith` names one person, `John Smith`. The `Mrs.` is an unnamed spouse, so there is no second row.

A title that governs a list is ignored while each named person is extracted (`Senators Warren and Booker` → `Warren` and `Booker`).

## Entity and synonyms

Strip attached titles, possessives, and sentence punctuation first (rules below). What remains is copied verbatim: original spelling, capitalization, and internal punctuation. Never capitalize a name yourself. Do not translate, expand, abbreviate, or reorder it.

- **`entity`** is the fullest attested form: the one with the most name tokens (given names, initials, surname, generational suffix). A quoted nickname does not count. If that same form appears with different casing, prefer a capitalized occurrence that is actually written in the source. If no capitalized occurrence exists, keep the lowercase form exactly as written. Never create a capitalized form. If equally full forms still tie, use the one that appears first.
- **`synonyms`** are the other spellings of that person. Do not include the entity string. Do not add aliases the source never writes. Emit name-forms first, then aliases. Name-forms are unique subsets, unique initialisms, and comma inversions. Aliases are explicit aliases and quoted nicknames. Within each group, keep first-appearance order. `Marcus Vance`, also called `"The Hound,"` then later `Vance` → `Vance;The Hound`.
- Deduplicate case-insensitively. When duplicates differ only by case, keep a capitalized source occurrence if one exists; otherwise keep the first source casing.
- Wrapping punctuation does not make a new spelling. `Ada`, `Ada.`, and `"Ada"` are one spelling; keep `Ada`. Periods inside initials are part of the spelling: `J.F.K.` and `JFK` are different.
- An inverted comma-form (`Lovelace, Ada`) is a different spelling. It stays a synonym unless it is the fullest form chosen as `entity`.

Examples:

- `ada lovelace` followed by `Ada Lovelace` → entity `Ada Lovelace`.
- Only `ada lovelace` appears → entity `ada lovelace`.
- `ada lovelace` and later `Ada` → entity `ada lovelace`, synonym `Ada`; the source never writes the full form as `Ada Lovelace`.

### What may be removed from a name

These are the only changes to an otherwise verbatim name:

- Attached titles and role labels are removed and discarded. Generational suffixes stay on the entity: `Jr.`, `Sr.`, `II`, `III`, `IV`.
- A trailing possessive `'s` or a trailing apostrophe used as a possessive. `Lovelace's` → `Lovelace`. `O'Brien's` → `O'Brien`. Keep apostrophes that belong to the name (`O'Brien`).
- A trailing period that ends a sentence. Keep periods that belong to initials or to a suffix that stays on the name (`J.F.K.`, `Jr.`).
- A quoted nickname, moved to `synonyms`.
- Wrapping quotation marks around an entire name.

`Sr.` is a generational suffix in `John Smith Sr.` and stays on the entity. `Sr.` is a courtesy title in `Sr. García` and is discarded.

## Ignore titles

Titles are context only. Never output them in `entity`, `synonyms`, or another column.

Strip a token only when it functions as a title or role on that mention:

- Before the name: `Dr. Martin Luther King Jr.`, `CEO Jane Smith`, `King Charles`, `Lord Byron`.
- After the name, set off as a suffix or office: `Jane Smith, PhD`, `Jane Smith, CEO`. Known suffixes may be stripped without a comma: `PhD`, `M.D.`, `MD`, `Esq.`, `RN`.
- In an appositive role: `Jane Smith, the CEO`, `Jane Smith, a mathematician`, `Jane Smith, director of research`.

Do not strip a token that is part of the name:

- Surname or given-name slot, including when the word is also a title word: `Stephen King`, `Martin Luther King Jr.` (`King` stays; `Jr.` stays).
- A leading word that is part of a stage name or pseudonym when the text does not refer to the person by the remainder alone: `Lady Gaga`, `Sir Mix-a-Lot`.
- If stripping would leave no name, keep the original mention when it is clearly a name (`Prince`), and skip the mention when it is only a role (`the CEO`).

Illustrative title words, not a closed list: `Mr.`, `Mrs.`, `Ms.`, `Miss`, `Mx.`, `Dr.`, `Prof.`, `Professor`, `Sir`, `Dame`, `Rev.`, `Hon.`, `Lord`, `Lady`, `Capt.`, `Col.`, `Gen.`, `Adm.`, `Rep.`, `Sen.`, `Amb.`, `Fr.`, `Rabbi`, `Imam`, `Pope`, `King`, `Queen`, `Prince`, `CEO`, `President`, `Senator`, `Governor`, `Judge`, `Director`. Whether the token functions as part of the name or as a title wins over the list.

## Output

Read and follow `format-as-csv/SKILL.md` to print the rows. That skill owns delimiter, line endings, empty cells, quoting, and quote escaping. **Do not write the CSV to disk.**

Column contract for that skill:

- Header, exact and unquoted: `entity,synonyms`. Emit it even when there are no people.
- `entity` is a required scalar. If it would be blank, do not emit the row.
- `synonyms` is one cell. A person with no synonyms has an empty cell. Several names are joined with `;` and no spaces, name-forms before aliases. Quote that cell only when `format-as-csv/SKILL.md` requires it.
- Keep every column. An empty `synonyms` is an empty last field: `Stephen King,`
- Unknown optional fields are empty.
- Emit rows in order of each person's first appearance.

## Worked example

Source text:

```
On March 12, 2024, Dr. Martin Luther King Jr. spoke in Washington. Stephen King signed books the same day. Robert "Bob" Smith arrived with Jane Smith, PhD, the CEO, and John Smith. Smith waved, and Bob waved back. The badge read "Smith, John". Mr. and Mrs. John Smith hosted Senators Warren and Booker. Lady Gaga performed. Lovelace's notes describe Ada Lovelace (also called Ada). Washington University sent no one. Marcus Vance, also called "The Hound," arrived after that. Vance left. She thanked the crew.
```

Required output, printed by `format-as-csv/SKILL.md` (each record ended by CRLF, including after the last row):

```
entity,synonyms
Martin Luther King Jr.,
Stephen King,
Robert Smith,Bob
Jane Smith,
John Smith,"Smith, John"
Warren,
Booker,
Lady Gaga,
Ada Lovelace,Lovelace;Ada
Marcus Vance,Vance;The Hound
```

`Dr.` is ignored and `Jr.` stays. `King` in both King names is a surname. `Bob` is the quoted nickname, then a later mention of the same person. Bare `Smith` matches three people, so it is dropped. `Smith, John` keeps its internal space and is a comma inversion of `John Smith`, not a new person. `Mrs.` names no one. `Senators`, `PhD`, `CEO`, and `a mathematician` are ignored. `Lady Gaga` stays intact. `Lovelace's` is the possessive of `Ada Lovelace`; `Ada` is an explicit alias, so the name-form `Lovelace` comes before it. `Vance` is a name-form and `The Hound` is an alias, so `Vance` comes first even though the nickname is written first. `Washington`, `Washington University`, `She`, and `the crew` are not people rows.

## Empty input

If the source contains no extractable people, emit the header only:

```
entity,synonyms
```

## Before replying

Check silently, then print only the CSV:

- The reply follows `format-as-csv/SKILL.md`. The header is exactly `entity,synonyms`.
- One row per person, in order of first appearance. `entity` is the fullest attested name, not the first short form.
- No synonym repeats `entity`. Synonyms list name-forms before aliases, and each group is in first-appearance order. No title, honorific, rank, office, profession, or role label is output.
- Every name keeps source-attested casing. A capitalized source occurrence wins over its lowercase duplicate; lowercase stays lowercase when no capitalized occurrence exists.
- No `UNKNOWN` or other sentinel. Empty optional fields are empty.
- Ambiguous short forms and unnamed people are absent.
- Full names were listed first, then the text was scanned for unique standalone surnames and supported nicknames before rows were compiled.
- Journalists, byline authors, and media staff named only in credits, headers, or `By` lines are absent. Subjects of the events are present.

## Do not

- Do not write the CSV to disk or wrap it in a markdown fence.
- Do not output JSON or a Markdown table.
- Do not extract places, organizations, events, or selectors.
- Do not leave `entity` blank or repeat it in `synonyms`.
- Do not add a row for a synonym, a pronoun, an unnamed role, or an ambiguous short form.
- Do not expand a name the source never writes.
- Do not output titles or role descriptions.
- Do not fabricate people or aliases.
- Do not extract journalists, byline authors, or media staff mentioned only in credits, headers, or `By` lines.
- Do not favor observers or reporters over the people involved in the events.

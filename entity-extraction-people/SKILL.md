---
name: extract-people
description: Extracts people from text into strict one-row-per-person CSV. Uses the fullest attested name as entity, keeps other source spellings as synonyms, and puts only functioning honorifics and job titles in title. Use when the user asks to extract people, persons, person names, or a people-only entity CSV.
---

# Extract people

Extract **people** from the given text. Use only names and titles written in that text. Do not add people, aliases, or titles from outside knowledge.

Return **only** the CSV as the assistant message, printed by following `format-as-csv/SKILL.md`. Do not add prose, markdown fences, or commentary unless the user explicitly asks for an explanation after the CSV.

**Do not write the CSV to disk.** Do not run a shell command, script, or tool to create, save, redirect, or append a file. Reading a file the user points at is fine. The result is the CSV printed in the reply.

## When to use

- The user provides text or a file and wants people extracted.
- The user asks for people, persons, or person names.
- The user wants spreadsheet-ready or machine-readable people CSV.

## Extraction

1. Read the full source text, or the file the user points at.
2. Extract named people only. Skip pronouns, unnamed groups, and roles with no personal name (`She`, `the crew`, `the CEO` alone).
3. A mention that refers to a person is extracted even if that string is also a place or organization somewhere in the text. Emit only the person row.
4. Do not extract a person merely because their name is embedded in another name. `Washington University` is not a person named `Washington`. Extract the person only when the text refers to them as a person.
5. Do not extract a person who appears only inside an email address, email domain, or URL host unless the text also refers to them as a person.
6. Group mentions of the same person onto one row (see below). If two full names might be the same person and the text does not show it, emit two rows.
7. Build `entity`, `synonyms`, and `title` from the grouped mentions, then emit rows in order of each person's first appearance. Print those rows by following `format-as-csv/SKILL.md`.

## Same person

Merge mentions into one row only when one of these is true:

- **Explicit alias.** The text equates the names (`also called`, `aka`, `née`, `born`, `whose real name is`, parentheses, or a quoted nickname inside the name). `Robert "Bob" Smith` is entity `Robert Smith` and synonym `Bob`. The quoted nickname is not part of `entity`.
- **Unique subset.** After title stripping, every name token of one mention sits inside exactly one other person's name, and no other person shares that token. `Lovelace` merges into `Ada Lovelace` when she is the only Lovelace.
- **Unique initialism.** A letter-initial form merges into a fuller name only when both appear and the initials match that one person. `JFK` merges into `John F. Kennedy` when both refer to the same person. If the full name is absent, keep `JFK` and do not expand it.
- **Comma inversion.** `Smith, John` is the same person as `John Smith` when the name tokens match one person.

Do not merge:

- Two full names that share only a given name or only a surname (`John Smith` and `Jane Smith`; `John Smith` and `John Doe`).
- A short form that matches more than one person. Drop it. Do not attach it to any row, and do not emit a row for the fragment (`Smith` when `Robert Smith`, `Jane Smith`, and `John Smith` are all present).
- Two people the model only knows are the same from outside knowledge.

`Mr. and Mrs. John Smith` names one person, `John Smith`. The `Mrs.` is an unnamed spouse, so there is no second row and `Mrs.` is not John's title.

A title that governs a list applies to each named person. If it is plural only because it covers the list, store the singular (`Senators Warren and Booker` → title `Senator` on each).

## Entity and synonyms

Strip titles, possessives, and sentence punctuation first (rules below). What remains is copied verbatim: original spelling, capitalization, and internal punctuation. Do not translate, expand, abbreviate, or reorder it.

- **`entity`** is the fullest attested form: the one with the most name tokens (given names, initials, surname, generational suffix). A quoted nickname does not count. If two forms are equally full, use the one that appears first. Keep that occurrence's casing.
- **`synonyms`** are the other spellings of that person. Do not include the entity string. Do not add aliases the source never writes. Emit name-forms first, then aliases. Name-forms are unique subsets, unique initialisms, and comma inversions. Aliases are explicit aliases and quoted nicknames. Within each group, keep first-appearance order. `Marcus Vance`, also called `"The Hound,"` then later `Vance` → `"Vance;The Hound"`.
- Deduplicate case-insensitively. Keep the first casing.
- Wrapping punctuation does not make a new spelling. `Ada`, `Ada.`, and `"Ada"` are one spelling; keep `Ada`. Periods inside initials are part of the spelling: `J.F.K.` and `JFK` are different.
- An inverted comma-form (`Lovelace, Ada`) is a different spelling. It stays a synonym unless it is the fullest form chosen as `entity`.

### What may be removed from a name

These are the only changes to an otherwise verbatim name:

- Functioning titles, moved to `title` (see below). Generational suffixes stay on the entity: `Jr.`, `Sr.`, `II`, `III`, `IV`.
- A trailing possessive `'s` or a trailing apostrophe used as a possessive. `Lovelace's` → `Lovelace`. `O'Brien's` → `O'Brien`. Keep apostrophes that belong to the name (`O'Brien`).
- A trailing period that ends a sentence. Keep periods that belong to initials or to a suffix that stays on the name (`J.F.K.`, `Jr.`).
- A quoted nickname, moved to `synonyms`.
- Wrapping quotation marks around an entire name.

`Sr.` is a generational suffix in `John Smith Sr.` and stays on the entity. `Sr.` is a courtesy title in `Sr. García` and moves to `title`.

## Titles

A title is a courtesy honorific, rank, professional suffix, or job the text attaches to that name. It is not a loose description.

Strip a token only when it is **functioning as a title on that mention**:

- Before the name: `Dr. Martin Luther King Jr.`, `CEO Jane Smith`, `King Charles`, `Lord Byron`.
- After the name, set off as a suffix or office: `Jane Smith, PhD`, `Jane Smith, CEO`. Known suffixes may be stripped without a comma: `PhD`, `M.D.`, `MD`, `Esq.`, `RN`.

Do not strip a token that is part of the name:

- Surname or given-name slot, including when the word is also a title word: `Stephen King`, `Martin Luther King Jr.` (`King` stays; `Jr.` stays).
- A leading word that is part of a stage name or pseudonym when the text does not refer to the person by the remainder alone: `Lady Gaga`, `Sir Mix-a-Lot`.
- If stripping would leave no name, keep the original mention when it is clearly a name (`Prince`), and skip the mention when it is only a role (`the CEO`).

Illustrative title words, not a closed list: `Mr.`, `Mrs.`, `Ms.`, `Miss`, `Mx.`, `Dr.`, `Prof.`, `Professor`, `Sir`, `Dame`, `Rev.`, `Hon.`, `Lord`, `Lady`, `Capt.`, `Col.`, `Gen.`, `Adm.`, `Rep.`, `Sen.`, `Amb.`, `Fr.`, `Rabbi`, `Imam`, `Pope`, `King`, `Queen`, `Prince`, `CEO`, `President`, `Senator`, `Governor`, `Judge`, `Director`. The function test wins over the list.

Put the stripped title in `title`, plus an office stated for that person (`the CEO` → `CEO`). Drop a leading `a`, `an`, or `the`.

Do **not** put incidental descriptions in `title`: `a mathematician`, `a resident of London`, `author of X`, `activist`, `mother of three`.

- An appositive office is a title even when it starts with `a`, `an`, or `the`, and even with a status word in front (`former`, `acting`, `interim`, `then`). Store the office only: drop the article and the status word. End the title before `who`, `at`, or `of` when that starts a biography, employer, or affiliation.
- Store `Mr.`, `Mrs.`, `Ms.`, `Miss`, and `Mx.` only when that person has no rank, professional suffix, or job. When one of those is present, strip the courtesy title from the name and leave it out of `title`.

Merge titles from every mention of that person onto the one row. Dedupe case-insensitively and keep the first spelling. Order honorifics and professional suffixes by appearance, then job roles by appearance. Do not put a title in `synonyms`. Do not invent a title.

One title is a scalar (`Dr.`). Several titles are a list with no spaces around the semicolons (`PhD` and `CEO` → `PhD;CEO`). `format-as-csv/SKILL.md` quotes that list.

## Output

Read and follow `format-as-csv/SKILL.md` to print the rows. That skill owns delimiter, line endings, empty cells, quoting, list joining, semicolon escaping, and quote escaping. **Do not write the CSV to disk.**

Column contract for that skill:

- Header, exact and unquoted: `entity,synonyms,title`. Emit it even when there are no people.
- `entity` is a required scalar. If it would be blank, do not emit the row.
- `synonyms` is a list column. A person with no synonyms has an empty cell.
- One `title` is a scalar. Several titles are a list.
- Keep every column. An empty `title` is an empty last field: `Ada Lovelace,"Lovelace;Ada",`
- Unknown optional fields are empty.
- Emit rows in order of each person's first appearance.

## Worked example

Source text:

```
On March 12, 2024, Dr. Martin Luther King Jr. spoke in Washington. Stephen King signed books the same day. Robert "Bob" Smith arrived with Jane Smith, PhD, the CEO, and John Smith. Smith waved, and Bob waved back. The badge read "Smith, John". Mr. and Mrs. John Smith hosted Senators Warren and Booker. Lady Gaga performed. Lovelace's notes describe Ada Lovelace (also called Ada). Washington University sent no one. Marcus Vance, also called "The Hound," arrived after that. Vance left. She thanked the crew.
```

Required output, printed by `format-as-csv/SKILL.md` (each line ended by `\n`, including after the last row):

```
entity,synonyms,title
Martin Luther King Jr.,,Dr.
Stephen King,,
Robert Smith,"Bob",
Jane Smith,,"PhD;CEO"
John Smith,"Smith, John",Mr.
Warren,,Senator
Booker,,Senator
Lady Gaga,,
Ada Lovelace,"Lovelace;Ada",
Marcus Vance,"Vance;The Hound",
```

`Dr.` is stripped and `Jr.` stays. `King` in both King names is a surname. `Bob` is the quoted nickname, then a later mention of the same person. Bare `Smith` matches three people, so it is dropped. `Smith, John` keeps its internal space and is a comma inversion of `John Smith`, not a new person. `Mrs.` names no one. `Senators` is stored as `Senator` on each named person. `Lady Gaga` stays intact. `Lovelace's` is the possessive of `Ada Lovelace`; `Ada` is an explicit alias, so the name-form `Lovelace` comes before it. `Vance` is a name-form and `The Hound` is an alias, so `Vance` comes first even though the nickname is written first. `Washington`, `Washington University`, `She`, and `the crew` are not people rows. `PhD` and `CEO` are titles; a loose description such as `a mathematician` would not be.

## Empty input

If the source contains no extractable people, emit the header only:

```
entity,synonyms,title
```

## Before replying

Check silently, then print only the CSV:

- The reply follows `format-as-csv/SKILL.md`. The header is exactly `entity,synonyms,title`.
- One row per person, in order of first appearance. `entity` is the fullest attested name, not the first short form.
- No synonym repeats `entity`. Synonyms list name-forms before aliases, and each group is in first-appearance order. No functioning title remains on `entity`. No incidental description is in `title`.
- No `UNKNOWN` or other sentinel. Empty optional fields are empty.
- Ambiguous short forms and unnamed people are absent.

## Do not

- Do not write the CSV to disk or wrap it in a markdown fence.
- Do not output JSON or a Markdown table.
- Do not extract places, organizations, events, or selectors.
- Do not leave `entity` blank or repeat it in `synonyms`.
- Do not add a row for a synonym, a pronoun, an unnamed role, or an ambiguous short form.
- Do not expand a name the source never writes.
- Do not fabricate people, aliases, or titles.

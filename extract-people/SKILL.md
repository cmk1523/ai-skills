---
name: extract-people
description: Extracts the people named in a text as a list of name strings, one per distinct person, in their fullest attested form. Ignores credits, titles, handles, places, and organizations. Use when the user asks to extract people or person names.
---

# Extract people

Return every distinct **person** named in the text as one plain string per person. The output is a flat list of strings. Never return objects, nested fields, aliases, or extra columns. Do not add people from outside knowledge.

Read the whole text before deciding any entry. Each named human gets exactly one entry: two entries for one person is an error, and so is zero.

## Credits are skipped (check this first)

A person named only in a credit or production line is NOT extracted, even when a job title, department, organization, or company follows the name. Credit markers include: `By ...`, `Reporting by`, `Additional reporting by`, `Photo(s)/Photograph(s)/Images/Video: ...`, `Edited by`, `Compiled by`, `Prepared by`, `Transcript prepared by`, `Reviewed by`, `Approved by`, `Source:`. This holds in long documents, and at the very start or end of the text.

Example: `Transcript prepared by: Nadia Quillfeather, Clearwater Records Unit` at the bottom of a hearing means `Nadia Quillfeather` is not extracted (unless she also takes part in the events).

Extract such a person only if they are also named elsewhere as taking part in the events or conversation. A sign-off in a conversation (an agent replying to a customer) is participation, not a credit. A header field naming a role-holder inside the events (`Incident commander: ...`, `Court Clerk: ...`, `For Plaintiff: ...`) is a participant, not a credit.

## What counts as a person

- Any mention that refers to a human being by a personal name: full names, a given name alone (`Bram`, `Lior`), or a surname alone (`Ostrander`).
- A lone surname with no fuller form anywhere in the text is still a person (`ask Ostrander`, `Ostrander signed the form`). Return it even when it is mentioned only in passing, only talked about by others, or when you cannot tell its given name or exact role. An unexplained capitalised word that acts like a human (signs, approves, replies, is away, has an assistant, is called he or she) is a person, not a team or company.
- Lowercase names are people. In informal writing, a lowercase word right after a relation or role word, or after `named` / `called`, is a name: `my manager noemi`, `my landlord otis`, `a woman named freya`, `thanks marisol`. If no fuller form exists, return it as written.
- A chat or transcript speaker label that is a personal name (`Dmitri:`, `Fiona Valdecasas:`) is a person.
- People who are only mentioned, not speaking, are people too: coworkers, bosses, family members, customers, support staff, assistants, and agents who sign off with a name.

## What is not a person

- Usernames, handles, `@mentions`, `u/` names, and email addresses or their local parts (`tori_k`, `devon.t`, `@hollis_q`, `b.varga@...`). Never turn a handle into a name (`hollis_q` is not `Hollis Q`). If the text also writes that person's real name, extract the real name.
- Service accounts, threat-group labels, pets, and anonymous roles (`the CEO`, `a contractor who declined to give a name`).
- Initial sign-offs such as `-QH`, and pronouns.
- Places, organizations, products, events, and software.
- A person's name inside the name of a venue, facility, institution, firm, or product is not a person (`Franklin University`, `Marlowe Auditorium`, `Dunmore Water Engineering`, `Sutter Lounge`). Only count that person if the text names them separately as a person.
- Names that appear only in case captions or legal citations (`X v. Y`).
- People who only report on or observe the events from outside.

## Same person: one entry per person

Compare mentions case-insensitively. Merge mentions into one entry when:

- **Short first, full later.** A person often appears first under a short name and only later under a fuller one: a speaker label `Zdenka` near the top and `Zdenka Hrubá` further down; `a guy named wim` and, later, `wim Aalders`. That is one person and one entry: the fuller form. Replace the short entry; never keep both.
- **Explicit alias.** The text equates the names (`also known as`, `aka`, `née`, `X is Y`, `full name Y`, parentheses, or a quoted nickname inside the name). `Alaric "Al" Smith` is one person.
- **Subset.** A mention whose name tokens all sit inside exactly one other person's name is the same person (`Lovelace`, `Ada`, `ada`, `C. Vandermeer` for `Cordelia Vandermeer`). This applies regardless of order, casing, or where the mention appears, and to speaker labels too: a speaker `Sana` and a speaker `Sana Draganov` are one person, and so are `pedro` and `pedro Alvarez`. A later bare surname is the same person when no other listed person shares it.
- **Nickname or short form.** `Bill`, `Mike`, and `ray` merge into a fuller name when the text supports it, in any of these ways:
  - an explicit alias;
  - exactly one listed person whose given name it is a usual short or pet form of, even when spelled differently (`Peggy` for `Margaret Thistlewood`);
  - a role or context link (`Vic, the technician` and a full-named technician; a staff member first mentioned by a short name, and a full-named person in that same role who later uses that short name for themselves).

  When such a link exists, the short name is not a new person.
- **Comma inversion.** `Tanaka, Hiroshi` is `Hiroshi Tanaka`.
- **Initialism.** `EMQ` or `Q. Lê` merges with the full name only when the match is unique.

**Never combine a nickname with a surname.** If the text says `Verena "Vee" Ashcombe`, or a person is called `Vee` later, the only entry is `Verena Ashcombe`. A quoted nickname standing next to the surname does not make `Vee Ashcombe` a name. Never output `Vee Ashcombe`, `Mike Tolliver`, or any `Nickname Surname` string unless those exact words appear together, with no quote marks, in the source. A nickname is never its own entry when it ties to a listed person; just drop it.

Do not merge:

- Full names that differ in the given name or the surname (`Elias Smith`, `Jane Smith`).
- Siblings, or a father and son (`Warren Fairbairn II`, `Warren Fairbairn III`). Suffixes distinguish people.
- A short form that could match more than one person (`Smith` when two Smiths exist). Drop it and emit no entry for it.
- People who are the same only according to outside knowledge.

A person who appears only under a short form (given name only, or surname only) is returned in that form.

## Which form to return

Return the fullest attested form of the name, copied from the source. Look across ALL mentions of the person, including later ones and answers such as "State your full name", and pick the form with the most name tokens (a form with a middle name beats a form without one). Never add tokens the source does not write, and never translate or expand a name yourself.

- Remove attached titles, honorifics, ranks, and role labels (`Dr.`, `Mr.`, `Ms.`, `Prof.`, `Capt.`, `Hon.`, `Mayor`, `Governor`, `Esq.`, `Uncle`, `Brother`, and appositive roles such as `, the CEO`). A title governing a list applies to each person.
- Keep generational suffixes (`Jr.`, `Sr.`, `II`, `III`).
- Remove possessive `'s`, sentence-ending periods, wrapping quotes, and a quoted nickname inside the name. Keep apostrophes that belong to the name and periods in initials.
- Keep a word that is part of a stage name or a surname even if it looks like a title. If stripping would leave nothing, keep the mention only if it is clearly a name.
- `Mr. and Mrs. Elias Smith` is one person, `Elias Smith`.

## Procedure

1. Mark credit lines (including the first and last lines of the text) and set those names aside.
2. Read the whole text to the end. List every full name, taking the longest form for each person.
3. Separate scanning pass: find given-name-only mentions, lowercase names (especially after relation words, `named`, `called`, `thanks`), surname-only mentions, speaker labels, and nicknames. For each, look for a listed full name that contains it as a token (any case), or that it is a nickname of. If exactly one does, it is that person: add nothing. If none does, it is a new person: add it as written. If several do, drop it.
4. Drop handles, credits, organizations, venues, places, and ambiguous fragments.
5. Final verification of every entry:
   - Its words appear in the source as those exact adjacent words (ignoring case, removed titles, and comma inversion). A quoted nickname beside a surname is not adjacency. If the check fails, fix or delete the entry.
   - Collision check, ignoring case: compare each entry with every other entry. If a one-word entry equals any word of a longer entry, delete the one-word entry. If two multi-word entries share a surname, keep both only when the source presents two different people; if one given name is the nickname of the other, keep only the formal one.
   - It is not from a credit line.
   - Recall: every lowercase name and every bare surname found in step 3 that has no fuller form is still in the list.
6. Output one string per person, in order of first appearance.

## Worked example

Text: `Prepared by: Mae Rindle, Hartwell Press. Dr. Tomas "Tom" Castellan met Lena Brandt at the Hartwell Auditorium. Tom said hi to his neighbor nico, and @lena_o answered. A clerk named wim took the form. Brandt, Lena left. Gus Hale Jr. arrived later; Hale waved. A rep from Orsen Labs said Perrin would call back. wim Aalders filed it, thanks wim.`

Output: `["Tomas Castellan","Lena Brandt","nico","wim Aalders","Gus Hale Jr.","Perrin"]`

- `Mae Rindle` is a credit.
- `Tom` is the nickname of `Tomas Castellan`. It is one entry, never `Tom Castellan` and never a separate `Tom`.
- `Hartwell Auditorium` is a venue, and `@lena_o` is a handle.
- `nico` is a lowercase given-name-only person with no fuller form.
- `wim` appears first alone and later as `wim Aalders`. That is one entry, the fuller form.
- `Brandt, Lena` is `Lena Brandt`.
- `Hale` merges into `Gus Hale Jr.`
- `Perrin` is a lone surname with no fuller form, so it is a person.

## Empty input

If there are no people, return an empty list.

## Do not

- Do not return aliases, nicknames, given names, or surnames as separate entries when they refer to an already listed person.
- Do not return credited-only people, titles, handles, organizations, venues, places, or products.
- Do not return objects or any shape other than a list of name strings.
- Do not invent names or expand names the source never writes.

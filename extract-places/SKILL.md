---
name: extract-places
description: Extracts named places from text, with country, city, and state filled only from words the text states verbatim. Use when the user asks to extract places or locations.
---

# Extract places

Return the distinct named places mentioned in the text, one entry per place. Each entry has the place's name plus `country`, `city`, and `state`. The output format is enforced elsewhere; this skill decides what to include, which form of the name to return, and what to copy into those three columns.

## What counts as a place

Include named:
- countries, states, provinces, counties, regions, cities, towns, villages, districts, neighborhoods
- continents, peninsulas, islands, valleys, deserts, basins
- rivers, lakes, seas, canals, mountains, capes, canyons
- ports and other geographic sites
- physical sites and landmarks such as bridges, parks, stations, forts, castles, squares, palaces and monuments. A named landmark counts even when its name has no type word (a fort called `Castillo de X`) and even when the type word is part of its name (`Harlow Bridge`).
- named regions with no type word (`the Vellmark`-style names), and fictional places from stories (`the Hollowmere`, `Castle Vandrel`).

Obscure names count too if the text uses them as places (`a map of Quillon`: a neighborhood, so include it). Lowercase or oddly cased names count (`brindle`, `calvera`), and so do names in all caps (a dateline such as `OSLO`).

These also count:
- A place used for its government or people (`Quenby rejected the plan`).
- A place used as a plain modifier before a lowercase common noun (`the Quenby office`, `The Brava ministry`, `the Tosca subsidiary`), including at the start of a sentence after "The". Only a capitalized official name (`Ministry of X`, `X Port Authority`) excludes the place.
- A place named only to say something is absent or negated (`no cases in Tavelia`, `denies travel to Dalmora`).
- A place after an address or street: `14 Elm Street, Wexcombe` still gives `Wexcombe`. Streets are skipped, the town is not.

## What to exclude

The key test: does the name refer to the location itself? If it is only a piece of the name of something else, skip it.

- **Generic place words** with no proper name: downtown, the city center, the harbor, the old market, the park, home, the lake house, the village.
- **Demonyms and adjectives** (Parisian, French, Texan, Hawaiian, Colombian, Swiss, Spanish). Do not extract the place they derive from. `Marlow-based` is different: it names Marlow directly, so include it.
- **Diseases and viruses named after places** (`X fever`, `X virus`, `X disease`) are not places.
- **Place names that are part of the name of a non-place thing.** Skip the place word in all of these, even if it is a famous city or country:
  - organizations, companies, banks, universities, sports teams, agencies, authorities, ministries, institutes, committees, subsidiaries, when the name is capitalized as one name (`University of X`, `Bank of X`, `X Dynamics`, `X Port Authority`, `Port of X Authority`, `X Maritime Institute`, `X Monetary Authority`, `X Pacific Rail`, `X Dominion`). Such a place is only extracted if it also appears elsewhere as a place on its own.
  - businesses named after places (`Hotel X`, `X Grill`, `X Lodge`, `X Cafe`, `X Bistro`). In "drinks at the Velmar Grill in Tosca", only Tosca is a place.
  - products, services, trains, routes and deals (`X Express`, `the X arrangement`, `the X line` as a brand)
  - geological or scientific formations and groups (`X River Basalt Group`)
  - hashtags and @handles, including any place inside them (`#City2030`, `@kim_from_X`).
- **Streets and addresses:** streets, roads, avenues, highways, routes and house numbers (`14 Elm Street`, `Harbor Road`, `Route 9`) are not places.
- **People's names:** a word that is a place name but is used as a person's first or last name, including names in credit lines (By, Photos, Prepared by, Reviewed by), is not a place.
- **Dates, emails, phone numbers, postal codes, and URL hosts** that are not used as places.

## One entry per place

- **Every level is its own entry, always.** In a comma hierarchy, a dateline, or an `of`/`in` container, the container (country, state, region) is a separate entry, never dropped and never combined into one string. Copying a container into another entry's column does NOT replace its own entry. The last item after a comma is a place too. `Quenby, Dalmora` gives two entries: `Quenby` and `Dalmora`. `Lund, Sweden` gives `Lund` and `Sweden`. `Dayton, Ohio` gives `Dayton` and `Ohio`. `the Moor region of Tavelia` gives `Moor` and `Tavelia`. `Kell, Pell County, Dalmora` gives three entries. `Brava, Ontaro-style province, Tavelia` also lists `Tavelia`. This holds even if the container is mentioned nowhere else.
- **Same place, several mentions:** return it once, in the **fullest** form that appears anywhere in the text. This covers:
  - short forms (`Mount Alder` then `Alder`; `Alder` then later `Alder National Park` gives only `Alder National Park`)
  - possessives (`Alder's slopes`)
  - abbreviations or initialisms of a name also given in full anywhere in the text. Return only the full name.
  - case-only variants (`calvera`, `Calvera`): return the capitalized form, even if the lowercase form came first or is more frequent.
  - a dateline in caps and a normal-case form (`OSLO, Norway` and later `Oslo`, or even only `OSLO`): return the normal capitalized form `Oslo`, never the all-caps one, in the name and in any column.
  - the same place repeated outside its hierarchy (`Dalmora` in `Quenby, Dalmora`, and later `the laws of Dalmora`): one entry.
- Two different places that share a name (`Salem, Fernwick` and `Salem, Massachusetts`) are separate entries.
- If the text uses only an acronym for a place and never the full name, return it as written.

## Form of each name

- Copy the name as written in the text, keeping spelling, accents, and punctuation. Do not translate or expand it.
- If a name appears only in lowercase (casual chat: `tarvos`, `lisboa`), return it exactly as written in lowercase, and write the same lowercase form in the columns. Capitalize only when a capitalized form of that same place appears somewhere in the text.
- Keep type words that are capitalized as part of the proper name: `Mount Alder`, `Kessel River`, `Lake Orrin`, `Port Varo`, `Cape of Storms`, `Elm Valley`, `Grand Union Station`, `Fort Brandt`, `Harlow Bridge`.
- Always drop a lowercase generic descriptor that follows the name (region, peninsula, province, area, island, city, county, district): `the Varga peninsula` gives `Varga`, `the Ruhn region` gives `Ruhn`, `the Vellmark-style region of X` gives `Vellmark-style`, `the Tesh area` gives `Tesh`. No entry may end in a lowercase word like "region" or "peninsula".
- Drop a lowercase descriptor before the name with `of`: `the island of Morwen` gives `Morwen`, `the town of Kell` gives `Kell`. A capitalized `of` name stays whole (`Cape of Storms`, `Gulf of Tessar`).
- Drop lowercase directional or qualifier words before a name (`northern`, `western`, `central`, `greater`, `upper`, `south of`, `near`): `northern Brava` gives `Brava`. Keep capitalized direction words that are part of the proper name (`North Veldmark`, `Southwest Ombria`).
- Drop leading articles and prepositions (`the`, `in`, `near`).

## Country, city, and state

Every entry has `country`, `city`, and `state`. Each value is a name copied from the text (in the same form and case as the name field), or `""`. Always include all three keys. Never infer containment from world knowledge: that a city is in some country, or a landmark is in some city, is not a reason to fill a column.

Fill a column only in these cases:

1. **The place itself is that kind of place.** This applies even with no context at all, and even after direction words are dropped (`northern Brava` still gives country `Brava` if Brava is a country). Decide the kind from what the place is, regardless of letter case.
   - A country fills `country` with its own name (`Brevonia` alone: country `Brevonia`). Never leave a country's own `country` empty.
   - A city, town or village fills `city` with its own name, even a lowercase or lesser-known one in casual chat (`tarvos`: city `tarvos`).
   - A US state fills `state` with its own name (`Nevada`, `Vermont`, `Ohio`), even when mentioned alone in a sentence. So does a unit written with the word State (`Tolan State`). Never leave a US state's own `state` empty.
   - Counties, provinces, historical or informal regions, large multi-country areas, and districts are NOT states and never fill `state` with their own name (`Marrowdale`-style county regions, `Southern Quillara`-style areas: all three columns empty unless a phrase states otherwise).
   - A city-state is both: it fills `country` and `city` with its own name (`Monaco`).
2. **The same phrase states one for this place.** Copy that name into the matching column:
   - a comma hierarchy. `Quenby, Dalmora`, Dalmora a state: Quenby gets `city` `Quenby`, `state` `Dalmora`. `Quenby, Tavelia`, Tavelia a country: Quenby gets `country` `Tavelia`. The container entry also fills its own kind (`Dalmora` gets `state` `Dalmora`; `Tavelia` gets `country` `Tavelia`).
   - a chain. `Kell, Pell County, Dalmora`: Kell gets `city` `Kell` and `state` `Dalmora`; Pell County gets `state` `Dalmora`; Dalmora gets `state` `Dalmora`.
   - a coordination closed by one place. `Quenby and Brava, Tavelia`: both get `country` `Tavelia`.
   - an `of` or `in` container. `the Moor region of Tavelia`: Moor's `country` is `Tavelia`.
   - a possessive city. `Quenby's Sable district`: Sable's `city` is `Quenby`.
   - a province or region written next to a country (`Karvel-style province, Country`) gets that `country`, with `city` and `state` empty.

Leave the column `""` in every other case.

- `city` is only for cities, towns and villages. A province, county, region, island, district, neighborhood, borough, valley, landmark or any other kind of place never gets `city` filled with its own name (`the Tesh district` has all three columns empty; a bare neighborhood name has `city` empty even if it is inside a city).
- A well-known province or region name is not a city even if it also sounds like one.
- A country, city, or state named in another sentence or clause, or in a dateline for a place elsewhere in the text, does not fill this place (`lives in Brava so maybe Tavelia after`: Brava's `country` stays empty). A city mentioned alone, or next to a demonym (`the Spanish ministry in Valdora`), has empty `country` and `state`. A landmark or river mentioned near a city has `city` empty.
- A demonym does not fill a column.
- Do not split a name. A state called `North Veldmark` has `state` `North Veldmark` and an empty `city`.
- Do not expand an abbreviation inside a column.
- If you are not sure what kind of place it is, leave that column empty.

## Order and empty input

List places in order of first appearance. If the text has no places, return an empty list.

## Before replying

- Is every entry a named place and not a generic word, demonym, disease, street, person's name, handle, or hashtag?
- Does each entry appear at least once as a place on its own, not only inside a capitalized organization, authority, institute, business, formation, or hashtag name? Remove it if not (e.g. a city inside `Port of X Authority` or `X Maritime Institute`).
- Go through every comma, dateline, and `of`/`in` hierarchy: does each country, state, and region in it have its own entry? Do all countries and US states, wherever they appear, have entries with their own column filled?
- Is any entry an abbreviation, shorter form, possessive, or descriptor-trimmed duplicate of another? Merge into the fullest form, using the normal capitalized form instead of ALL CAPS, in both name and columns.
- Does any entry keep a lowercase descriptor ("region", "peninsula") or qualifier ("northern")? Remove it. Did a lowercase-only name stay lowercase?
- Did you keep towns found after street addresses, lowercase-modifier places (`The X ministry`), obscure neighborhoods, and named landmarks and regions?
- Is every column either `""` or a name written in the same phrase? Clear any filled from knowledge, another sentence or clause, a demonym, or a dateline.
- Is `city` filled only for cities, towns and villages, and `state` only for US states and `X State` units (or a state named in the phrase)?

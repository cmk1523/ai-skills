---
name: extract-selectors
description: Extracts selectors (emails, phones, IP addresses, URLs, domains, MAC addresses, IMEIs, IMSIs, SSNs, credit card numbers, UUIDs, handles, usernames, hashtags, license plates, license IDs, API keys, and other labeled identifiers) from text into strict CSV, with a description of who or what each selector belongs to. Use when the user asks to extract selectors or identifiers.
---

# Extract selectors

Extract **selectors** from the given text: emails, phone numbers, IP addresses, URLs, domains, MAC addresses, IMEIs, IMSIs, social security numbers, credit card numbers, UUIDs, social handles, usernames, hashtags, license plates, license IDs, API keys, and any other explicit identifier that fits none of those types.

Selectors are identifiers, not ordinary nouns. Return **only** the CSV as the assistant message, printed by following `format-as-csv/SKILL.md`. Do not add prose, markdown fences, or commentary unless the user explicitly asks for an explanation after the CSV.

**Do not write the CSV to disk.** Do not run a shell command, script, or tool to create, save, redirect, or append a file. Reading a file the user points at is fine. The result is the CSV printed in the reply.

## When to use

- The user provides text or a file and wants selectors extracted.
- The user asks for selectors or identifiers.
- The user wants spreadsheet-ready or machine-readable selector CSV.

## Extraction

1. Read the full source text, or the file the user points at.
2. Extract selectors only.
3. Copy each `entity` **verbatim** from the source. Keep the original spelling, capitalization, and internal punctuation and spacing. Do not rewrite, expand, abbreviate, translate, or otherwise normalize the `entity` value. Strip surrounding sentence punctuation that is not part of the selector (a trailing period or comma, or enclosing parentheses, brackets, or quotes). Keep punctuation that belongs to the selector itself, such as the parentheses in `(415) 555-0100`.
4. Extract the selector as written. Do not split an email into a domain, or a URL into a host, domain, or path, unless that shorter selector also appears on its own in the text.
5. Do not extract a person, place, organization, or event that appears only inside an email, email domain, or URL host. Extract the selector itself.
6. Deduplicate case-insensitively on `entity`. Keep the first spelling that appears. Do not emit another row for a case-only variant. Spellings that differ by punctuation, spacing, or formatting each get their own row. Emit rows in order of first appearance. Print those rows by following `format-as-csv/SKILL.md`.
7. `selector type` is required. Use a specific type when one fits. If the value is clearly an identifier and no specific type fits, use `GENERIC`. If it is not clearly an identifier, do not emit the row.
8. `description` is required. Write a short phrase for who or what the selector belongs to, using only what the source states. When the text assigns it to a person, use `the [kind] for [person name]`, such as `the phone number for Ada Lovelace`. Resolve a clear pronoun to that person. When the text names no owner, describe the role it does state, such as `the press contact phone number`. Do not invent a person, organization, or purpose. Do not copy the `entity` value into the description. Do not end the phrase with a period.

### Selector types

`selector type` is exactly one of these scalar values:

`EMAIL`, `PHONE`, `IP_ADDRESS`, `URL`, `DOMAIN`, `MAC_ADDRESS`, `IMEI`, `IMSI`, `SSN`, `CREDIT_CARD`, `UUID`, `HANDLE`, `USERNAME`, `HASHTAG`, `LICENSE_PLATE`, `LICENSE_ID`, `API_KEY`, `GENERIC`

Dates, times, currency amounts, measurements, postal codes, and version numbers are not selectors. Do not use `GENERIC` for them.

- `EMAIL` — `press@example.com`
- `PHONE` — `+1-415-555-0100`
- `IP_ADDRESS` — `203.0.113.10`
- `URL` — `https://example.com/press`
- `DOMAIN` — `analyticalengine.org` when it appears as its own selector, not only inside an email or URL
- `MAC_ADDRESS` — `00:1A:2B:3C:4D:5E`
- `IMEI` — `490154203237518`
- `IMSI` — `310150123456789` when the text labels it as an IMSI. An unlabeled 15-digit number is not an IMSI.
- `SSN` — `078-05-1120`
- `CREDIT_CARD` — `4111 1111 1111 1111`. Keep the written grouping and separators. Do not extract an IMEI, IMSI, or SSN as a credit card.
- `UUID` — `550e8400-e29b-41d4-a716-446655440000`
- `HANDLE` — `@adalovelace`. A value that begins with `@` is `HANDLE`, not `USERNAME`. Do not strip the `@` to emit a username.
- `USERNAME` — `ada_lovelace` when the text calls that value a username, login, screen name, or account name. Do not include the label. A person's name is not a username unless the text uses one of those labels for that exact value.
- `HASHTAG` — `#AnalyticalEngine`
- `LICENSE_PLATE` — `ABC-1234` when the text calls it a license plate, plate, or tag. Keep hyphens and spaces that belong to the plate. Do not extract a state name or an unlabeled alphanumeric string.
- `LICENSE_ID` — `D1234567` when the text calls it a license or permit number, such as a driver's license or professional license. A vehicle plate is `LICENSE_PLATE`, not `LICENSE_ID`.
- `API_KEY` — `sk_test_analytical_engine` when the text calls it an API key, access key, secret, or token, or when the value itself begins with a recognizable key prefix such as `sk_live_`, `sk_test_`, `AKIA`, `ghp_`, or `xoxb-`. Copy the value verbatim and do not extract the label.
- `GENERIC` — `8839102` when the text labels it as an identifier that fits no specific type, such as an account number, passport number, employee ID, or badge number. Do not include the label. Do not use `GENERIC` when a specific type fits.

License plates, license IDs, usernames, and other non-standard identifiers have no single written form. Extract a plate, license ID, username, or `GENERIC` identifier only with a label in the text. An API key may instead use a recognizable key prefix. Skip an unlabeled string.

## Output

Read and follow `format-as-csv/SKILL.md` to print the rows. That skill owns delimiter, line endings, empty cells, quoting, and quote escaping. **Do not write the CSV to disk.**

Column contract for that skill:

- Header, exact and unquoted: `entity,selector type,description`. Emit it even when there are no selectors.
- `entity`, `selector type`, and `description` are required scalars. If any would be blank, do not emit the row.
- `description` is a short phrase with no trailing period.
- `selector type` is one token from the allowed list.
- Keep every column.
- Emit rows in order of first appearance.

## Worked example

Source text:

```
On March 12, 2024, Dr. Ada Lovelace (also called Ada), a mathematician, spoke at the Analytical Engine Summit in London, England, United Kingdom. She also visited Washington, D.C. in the US. Contact press@example.com or +1-415-555-0100. The summit ran through March 14, 2024. NASA, ACME Labs, and Amazon hosted the stream from 203.0.113.10. A crew filmed along the Amazon. Follow @adalovelace and #AnalyticalEngine. Her username was ada_lovelace. The schedule is at https://example.com/press. Updates are posted on analyticalengine.org. Logged identifiers included SSN 078-05-1120, IMEI 490154203237518, IMSI 310150123456789, card 4111 1111 1111 1111, MAC 00:1A:2B:3C:4D:5E, and UUID 550e8400-e29b-41d4-a716-446655440000. Her license plate was ABC-1234, her driver's license was D1234567, and the booth API key was sk_test_analytical_engine. Her account number was 8839102. Call again at (415) 555-0100 or PRESS@example.com.
```

Required output, printed by `format-as-csv/SKILL.md` (each record ended by CRLF, including after the last row):

```
entity,selector type,description
press@example.com,EMAIL,the press contact email
+1-415-555-0100,PHONE,the press contact phone number
203.0.113.10,IP_ADDRESS,the IP address the stream was hosted from
@adalovelace,HANDLE,the social handle for Ada Lovelace
#AnalyticalEngine,HASHTAG,the hashtag mentioned with Ada Lovelace's handle
ada_lovelace,USERNAME,the username for Ada Lovelace
https://example.com/press,URL,the URL for the schedule
analyticalengine.org,DOMAIN,the domain where updates are posted
078-05-1120,SSN,the logged Social Security number
490154203237518,IMEI,the logged IMEI
310150123456789,IMSI,the logged IMSI
4111 1111 1111 1111,CREDIT_CARD,the logged credit card number
00:1A:2B:3C:4D:5E,MAC_ADDRESS,the logged MAC address
550e8400-e29b-41d4-a716-446655440000,UUID,the logged UUID
ABC-1234,LICENSE_PLATE,the license plate for Ada Lovelace
D1234567,LICENSE_ID,the driver's license for Ada Lovelace
sk_test_analytical_engine,API_KEY,the API key for the booth
8839102,GENERIC,the account number for Ada Lovelace
(415) 555-0100,PHONE,the phone number to call again
```

The periods after `+1-415-555-0100`, `#AnalyticalEngine`, `ada_lovelace`, `https://example.com/press`, `analyticalengine.org`, `sk_test_analytical_engine`, `8839102`, and `PRESS@example.com` are sentence punctuation, so none of them is part of the selector. `PRESS@example.com` differs from `press@example.com` only by case, so it is not a second row. `(415) 555-0100` differs by punctuation, so it is its own row. `example.com` is not a separate row because it appears only inside the email and the URL. `analyticalengine.org` is a `DOMAIN` row because it appears on its own. `310150123456789` is an `IMSI` because the text labels it. `4111 1111 1111 1111` keeps its internal spaces. `ABC-1234`, `D1234567`, and `sk_test_analytical_engine` are extracted because the text labels them as a license plate, a driver's license, and an API key. The labels themselves are not part of the entity. `8839102` is `GENERIC` because the text labels it as an account number and no specific type fits. `ada_lovelace` is a `USERNAME` because the text calls it a username. `Ada` and `Ada Lovelace` are not usernames. `@adalovelace` stays a `HANDLE`; the `@` is not stripped to make a second row. Ada Lovelace, the summit, the places, NASA, ACME Labs, Amazon, and the dates are not selector rows. Each `description` names the role the text gives the selector and the owner when the text assigns one. `the username for Ada Lovelace` resolves "her" to Ada Lovelace. The press email and press phone are not described as Ada's, because the text only says to contact those addresses. The logged identifiers are not given an owner.

## Empty input

If the source contains no extractable selectors, emit the header only:

```
entity,selector type,description
```

## Before replying

Check silently, then print only the CSV:

- The reply follows `format-as-csv/SKILL.md`. The header is exactly `entity,selector type,description`.
- Every `description` is a non-empty phrase with no trailing period. It names an owner only when the source assigns one.
- Case-only variants share one row (first spelling). Forms that differ by more than case each have a row, in order of first appearance. `entity` is the verbatim source spelling, without surrounding sentence punctuation.
- `selector type` is one allowed token and is never blank.
- No people, places, organizations, events, dates, times, currency amounts, measurements, postal codes, version numbers, or group labels.
- No `UNKNOWN` or other sentinel.

## Do not

- Do not write the CSV to disk or wrap it in a markdown fence.
- Do not output JSON, a Markdown table, or the five-group entity CSV.
- Do not extract people, places, organizations, or events.
- Do not extract dates, times, currency amounts, measurements, postal codes, or version numbers.
- Do not leave `entity`, `selector type`, or `description` blank.
- Do not rewrite, expand, abbreviate, or reformat a selector in the `entity` column. Do not keep surrounding sentence punctuation on `entity`.
- Do not split an email or URL into a shorter selector unless that shorter form also appears in the text.
- Do not give a case-only variant its own row.
- Do not add a `synonyms` column or any column besides `entity`, `selector type`, and `description`.
- Do not invent a person, organization, or purpose in `description`. Do not copy the `entity` value into `description`. Do not end `description` with a period.
- Do not write `UNKNOWN` or any other sentinel for a missing field.
- Do not fabricate selectors. Use `GENERIC` only for a labeled identifier that fits no specific type. If a value is not clearly an identifier, skip the row.
- Do not use `GENERIC` when a specific type fits.
- Do not extract an unlabeled alphanumeric string as a license plate, license ID, username, or `GENERIC` identifier. Do not extract a person's name as a username unless the text labels that exact value as a username, login, screen name, or account name. Do not strip `@` from a handle to create a username. Do not extract an API key that has neither a label nor a recognizable key prefix.

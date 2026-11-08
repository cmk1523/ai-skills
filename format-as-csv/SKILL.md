---
name: format-as-csv
description: >-
  Prints tabular data as strict RFC 4180 CSV in the reply and returns only that
  CSV. Use when the user asks to output, print, or format data as CSV, strict
  CSV, or spreadsheet-ready rows in the chat with no surrounding prose.
---

# Format as CSV

Return **only** the raw CSV as the assistant message:
- **No prose, labels, comments, or markdown code fences.**
- **Do not write to disk.** Do not run any tool, script, or shell command to create or modify a file (reading an input file the user points at is fine).
- **Do not alter facts.** Copy values verbatim (original spelling, casing, internal spaces, leading/trailing zeros, and scientific notation).

## Structure & Header

- **Delimiter:** Comma (`,`) only, with no space inserted after commas and no trailing comma after the last field.
- **Line endings:** CRLF (`\r\n`) after every record, including the last, as in RFC 4180. No blank lines before or between rows.
- **Multiple tables:** Only if the user explicitly asks for multiple tables, separate them with a single blank line and no labels.
- **Header (always emitted as the first row, even when there are 0 data rows):** Use the first source available:
  1. Column names the user stated, in that exact spelling and order.
  2. The header of an input CSV or table.
  3. Object keys in order of first appearance across records (omit keys that never appear).
  4. `value` (for a flat list of scalars or empty input with no column names).
- **Row order & filtering:** Keep input order unless asked to sort. Skip any row where all cells are empty, or where a column the user marked required is empty. Every row must have the exact same number of fields as the header.

## Cells & Quoting

RFC 4180 quoting only. Quote a field when its text contains a comma (`,`), a double quote (`"`), or a line break (`\n` or `\r`). Leave every other field unquoted. An apostrophe (`'`), a semicolon (`;`), and leading or trailing spaces do not trigger quoting. The same rule applies to every field, including the header.

- **Empty / missing values:** Write nothing (`a,,c` or `a,b,`). Never write `null`, `None`, `N/A`, `UNKNOWN`, or `""`. A whitespace-only string is data; write those spaces and leave the field unquoted. Do not trim spaces. Spaces are part of the field.
- **Escaping:** In a quoted field, double each embedded double quote (`"` → `""`). Never use backslash escapes (`\"`, `\n`, `\uXXXX`) or HTML entities. Keep a literal line break inside the quoted field.
- **No list encoding.** RFC 4180 has no array type, no `;` separator, and no `;;` escape. Do not quote a field because the value came from a list, and do not escape `;`. If the user already supplied the cell text, apply the quoting rule to that text. If a value is still an array or object and the user did not say how to render it, write compact single-line JSON as the cell text, then apply the quoting rule. `["Lovelace","Ada"]` contains commas and quotes, so the field is `"[""Lovelace"",""Ada""]"`.
- **Other JSON types:** Booleans as `true` or `false`. Numbers as their verbatim numerals.

## Worked example

Input records (`header: name,aliases,city,note,score`). Each `aliases` value below is already cell text:
1. Ada Lovelace; aliases `Lovelace;Ada`; city `London`; no note; score `-3`
2. Jane Smith; no aliases; no city; note `CEO, acting`
3. Name `Smith, John`; city `Boston`
4. Name `Robert "Bob" Smith`; aliases `Bob`
5. Name `=total`; note `=sum`
6. Name `O'Brien`; city `Dublin`

Required output (raw text in the reply, no code fence):

name,aliases,city,note,score
Ada Lovelace,Lovelace;Ada,London,,-3
Jane Smith,,,"CEO, acting",
"Smith, John",,Boston,,
"Robert ""Bob"" Smith",Bob,,,
=total,,,=sum,
O'Brien,,Dublin,,


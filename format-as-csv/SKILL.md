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
- **Line endings:** `\n` (LF), including after the last row. No blank lines before or between rows.
- **Multiple tables:** Only if the user explicitly asks for multiple tables, separate them with a single blank line and no labels.
- **Header (always emitted as the first row, even when there are 0 data rows):** Use the first source available:
  1. Column names the user stated, in that exact spelling and order.
  2. The header of an input CSV or table.
  3. Object keys in order of first appearance across records (omit keys that never appear).
  4. `value` (for a flat list of scalars or empty input with no column names).
- **Row order & filtering:** Keep input order unless asked to sort. Skip any row where all cells are empty, or where a column the user marked required is empty. Every row must have the exact same number of fields as the header.

## Cells & Quoting

- **Empty / missing values:** Write nothing (`a,,c` or `a,b,`). Never write `null`, `None`, `N/A`, `UNKNOWN`, or `""`. (A whitespace-only string is non-empty data; quote it.)
- **Scalars:** Leave unquoted unless the value contains a comma (`,`), double quote (`"`), single quote (`'`), line break (`\n` or `\r`), or leading/trailing whitespace. A single quote does not need further escaping; wrap the whole cell in `"` (`O'Brien` → `"O'Brien"`). Apply this to every field, including the header.
- **Lists / arrays:** Drop empty or `null` items, escape any literal `;` inside an item as `;;`, join items with `;` (no spaces around `;`), and **always quote the cell**—even for a single item (`"Ada"`, `"Lovelace;Ada"`, `"a;;b;c"`). An empty list is an empty unquoted cell. Treat a single value as a scalar unless the input provides it as an array/list or the user defines the column as a list.
- **Escaping:** Double any embedded double quote (`"` → `""`) and wrap the cell in `"`. Never use backslash escapes (`\"`, `\n`, `\uXXXX`) or HTML entities. Keep literal line breaks inside the quoted cell.
- **Other JSON types:** Booleans as `true` or `false`; nested objects as compact single-line JSON, quoted under the scalar rules.

## Worked example

Input records (`header: name,aliases,city,note,score` where `aliases` is a list):
1. Ada Lovelace; aliases `["Lovelace", "Ada"]`; city `London`; no note; score `-3`
2. Jane Smith; no aliases; no city; note `CEO, acting`
3. Name `Smith, John`; city `Boston`
4. Name `Robert "Bob" Smith`; aliases `["Bob"]`
5. Name `=total`; note `=sum`
6. Name `O'Brien`; city `Dublin`

Required output (raw text in the reply, no code fence):

name,aliases,city,note,score
Ada Lovelace,"Lovelace;Ada",London,,-3
Jane Smith,,,"CEO, acting",
"Smith, John",,Boston,,
"Robert ""Bob""Smith","Bob",,,
=total,,,=sum,
"O'Brien",,Dublin,,


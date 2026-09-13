---
name: rowguard-validate-csv
description: Validate a UTF-8 CSV file against a column schema before importing it into a database or automation workflow, using the RowGuard CSV Validation API.
api: RowGuard CSV Validation API
generated: '2026-09-13'
method: generated
source: openapi/rowguard-openapi.json
operations:
  - validateCsv
  - getExample
  - health
---

# Validate a CSV before import with RowGuard

Catch missing values, type errors, out-of-range numbers, disallowed values,
duplicate identifiers, and spreadsheet-formula (CSV injection) risks in one
HTTP request — before the rows reach your database or automation.

## Prerequisites

- A RapidAPI subscription to RowGuard (free BASIC tier = 25 requests/month).
- Send requests to the RapidAPI gateway with both `X-RapidAPI-Key` and
  `X-RapidAPI-Host` headers. Never send a key directly to the workers.dev origin —
  it accepts only gateway-authenticated calls.

## Steps

1. (Optional) Fetch a ready-made request body with `getExample`
   (`GET /v1/example`, no auth) to see the exact request shape.
2. Build a `ValidationRequest`: put your CSV text in `csv` (UTF-8, <=1,000 data
   rows, 50 columns, 256 KiB), and describe each column in `schema[]`
   (`name`, `type` one of string/integer/number/boolean/date/email, plus
   `required`, `unique`, `enum`, `min`, `max`, `max_length`).
3. Choose behavior flags as needed: `delimiter` (`,` `;` tab `|`), `trim`,
   `allow_extra_columns`, `formula_policy` (`reject` or `warn`),
   `include_data` (return passing rows), `max_errors` (1–200).
4. Call `validateCsv` (`POST /v1/validate`).
5. Read the `ValidationResult`: `valid` (boolean), `summary`
   (total/checked/valid/invalid rows, error/warning counts, `header_valid`),
   and `errors[]` (each with `code`, `message`, `row`, `column`, `end_line`,
   `severity`). A completed validation of a *bad* CSV still returns **HTTP 200** —
   inspect `valid`/`summary`/`errors`, do not rely on the status code.
6. If `include_data=true`, `data[]` holds the rows that passed (duplicate
   occurrences excluded).

## Error and reliability rules

- Transport errors use a custom envelope `{ "error": { "code", "message" },
  "request_id" }` with statuses 400/401/408/413/415/422/429/500/503.
- Every response carries a `request_id` (also the `X-Request-Id` header) — log it
  for support.
- On `429` or `503`, honor the `Retry-After` header before retrying. The call is
  stateless and stores nothing, so retries are safe.
- `getExample` and `health` need no auth; `validateCsv` requires the gateway key.

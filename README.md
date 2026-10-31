# SmartDataAI Sample Data — a synthetic messy-leads file, and its cleaned twin

The synthetic sample dataset used to demo SmartDataAI's free data-cleaning
tools — a small leads CSV with the problems real exports always have
(duplicates, missing values, four date formats, phone numbers in a dozen
formats), published next to the cleaned version so you can see exactly what
"clean" means, rule by rule.

**Synthetic, fictional, and safe for demos and tests** — that is the point of
this repository. Realistic dirty data is hard to practice on: you cannot use
real customer exports in a tutorial, a workshop, or a test suite. This file
you can — share it, screenshot it, and wire it into CI, without ever touching
a real person's data.

Run the messy file through the tools yourself and compare your result with
ours:

- [CSV Cleaner](https://smartdataai.ai/tools/csv-cleaner) — the free tool this sample was built for
- [Duplicate Remover](https://smartdataai.ai/tools/duplicate-remover) — step 4 below is its whole job
- More tools and guides at [smartdataai.ai](https://smartdataai.ai)

> **Synthetic data — no real people.** Every name, company, and email in
> this repository is invented. Emails use the reserved `example.com` /
> `example.org` / `example.net` domains, and phone numbers use the
> 555-0100–0199 range reserved for fictional use. Do not "correct" them
> into real addresses.

## What's inside (`data/`)

| File | Rows | What it is |
| --- | --- | --- |
| `leads-messy.csv` | 26 | The "before": a leads export with deliberate, catalogued problems |
| `leads-clean.csv` | 21 | The "after": the same leads after the cleaning rules below |

Columns (both files): `first_name, last_name, email, phone, company,
job_title, city, state, country, lead_source, created_date`.

## The mess catalog

Everything wrong with `leads-messy.csv`, on purpose:

| # | Problem | Where | Cleaning rule |
| --- | --- | --- | --- |
| 1 | Exact duplicate rows | Sarah Johnson ×2, Robert Taylor ×2 | Drop the copies — 2 rows removed |
| 2 | Same person, typed twice in different formats | John Smith, Maria Garcia, Emily Nguyen | Merge into one row, keeping the most complete version — 3 rows merged |
| 3 | Invalid emails | `james.wilson@@example.com`, `michael.brown@example` (no TLD), `tom.baker at example.com` | Blank the cell, keep the row — never guess a correction |
| 4 | Missing values | Email (Sarah Johnson), phone (Priya Patel), company (George Hall), job title (one Maria Garcia copy) | Leave blank; a blank you know about beats a value you invented. The title reappears via the merge in #2 |
| 5 | Phone chaos | `(415) 555-0132`, `415.555.0161`, `+1 (917) 555-0155`, `1-800-555-0147`, `+44 20 7946 0301`, … | Normalize to E.164: `+14155550132` |
| 6 | Date chaos | `2026-01-05`, `01/05/2026`, `Jan 12 2026`, `5-Jan-26`, `25-Mar-26` | Normalize to ISO `YYYY-MM-DD` |
| 7 | Case & whitespace chaos | `MARIA GARCIA`, `vp sales`, `  Daniel  ` / ` Kim `, `John.Smith@Example.COM`, `ACME ROBOTICS` | Trim; Title Case names, cities, titles; lowercase emails |
| 8 | Geography variants | Country: `US` / `USA` / `United States` / `U.S.A.` / `U.S.`; State: `CA` / `California` / `Ca` / `Il` / full names | Country → ISO alpha-2 (`US`, `GB`); US state → 2-letter code |
| 9 | Source-label variants | `LinkedIn` vs `linkedin` vs `Linked In` | Normalize to one label per source |

## Before → after, in numbers

- **26 rows → 21 rows** (2 exact duplicates dropped, 3 near-duplicates merged)
- 3 invalid emails blanked, 0 emails "fixed" by guessing
- Every remaining phone in one format (E.164); every date in ISO format
- Every email lowercase and syntactically valid — or honestly blank
- No row deleted for missing data; missing stays visibly missing

A spot check — the two John Smith rows:

```csv
# before (two rows)
john,smith,john.smith@example.com,415-555-0132,ACME ROBOTICS,vp sales,San Francisco,California,USA,linkedin,01/05/2026
John,Smith,John.Smith@Example.COM,(415) 555-0132,Acme Robotics,VP Sales,San Francisco,CA,US,LinkedIn,2026-01-05

# after (one row)
John,Smith,john.smith@example.com,+14155550132,Acme Robotics,VP Sales,San Francisco,CA,US,LinkedIn,2026-01-05
```

## The cleaning rules, in order

1. **Trim and normalize case** — whitespace first, then Title Case for
   names/cities/titles, lowercase for emails.
2. **Standardize formats** — phones to E.164, dates to ISO, country to ISO
   alpha-2, US states to their 2-letter code, source labels to a fixed list.
3. **Validate** — an email must have exactly one `@`, a real TLD, and no
   spaces; failures are blanked, not repaired by guessing.
4. **Deduplicate** — exact rows first; then match on normalized email, or
   on normalized phone, and merge keeping the most complete record.
5. **Keep the gaps visible** — missing values stay blank in the output.
   Cleaning is not enrichment.

## Try it yourself

1. Download [`data/leads-messy.csv`](data/leads-messy.csv).
2. Run it through the [CSV Cleaner](https://smartdataai.ai/tools/csv-cleaner)
   (and the [Duplicate Remover](https://smartdataai.ai/tools/duplicate-remover)
   if you want to isolate step 4).
3. Diff your output against [`data/leads-clean.csv`](data/leads-clean.csv).
   Any difference should be explainable by the rules above — if it isn't,
   open an issue and tell us which rule you read differently.

## License

MIT — see [LICENSE](LICENSE). The dataset is synthetic and may be reused
freely for demos, tests, and tutorials.

# Audit inputs for `work_index.json`

`work_index.json` and `manifest.json` are generated from `raw/` by the Morcus
Net crawler (`src/scripts/thelatinlibrary/crawl/run_crawl.ts`). These files are
the hand-checked inputs it reads besides `raw/`. Rebuild the index offline with:

```sh
THELATINLIBRARY_ROOT=/path/to/this/repo \
  npm run tsnp src/scripts/thelatinlibrary/crawl/run_crawl.ts -- --index-only
```

## `verified_audit_v2.json`

A one-off coverage audit (October 2026) matching every work on The Latin
Library against the curated Morcus Library (Perseus, Hypotactic and PHI
texts). The `summary` and `false_positives_corrected` keys record totals and
corrections made during the audit and are not read by the crawler.

Top-level keys holding works, each an array of entries:

| Key | Meaning | Index status |
| --- | --- | --- |
| `present_same_works` | In the Library under the same name | `in_library` |
| `different_name_crosswalk` | In the Library under another name | `in_library` |
| `partial_coverage_works` | Partly in the Library | `partial` |
| `classical_and_late_antique_missing` | Not in the Library | `candidate` |
| `patristic_christian_missing` | Not in the Library | `candidate` |
| `medieval_works` | Not in the Library | `candidate` |
| `neolatin_works` | Not in the Library | `candidate` |

A `candidate` or `partial` work becomes `excluded` when any of its pages is
excluded by the provenance scan (post-1930 edition or copyright notice).

Fields of an entry read by the crawler:

- `ll_author`, `ll_title`: display author and title.
- `ll_paths`: the work's pages, as `.txt` paths relative to the site root
  (`caesar/gall1.txt`); the crawler maps them to the `.html` / `.shtml` page in
  `raw/`.
- `era`, `source_shard`: used for the period of `classical_and_late_antique_missing`
  and the `in_library` / `partial` buckets.
- `perseus_id`, `hypotactic_id`, `phi_json_id`, `perseus_author`,
  `perseus_title`: the matching Library work (`libraryWorkIds`, and the link in
  `partialCoverage`).
- `notes`: shown as the `partialCoverage` note.
- `redundant_files`, `alternate_edition_files`: pages left out of `pages` and
  listed in `redundantOrAlternatePages` (strings or `{ "path": ... }`).
- `ll_words`: word count, used only when the pages can't be read.

Other fields (`status`, `perseus_words`, `in_hypotactic`, ...) are audit notes.

## `ll_file_duplicates.json`

Pairs of Latin Library files where one (`smaller`) is mostly contained in the
other (`larger`), found by token overlap during the same audit. When
`same_audit_entry` is true and the entry has more than one page, `smaller` is
moved to `redundantOrAlternatePages`. `containment_of_smaller`, the token
counts and the `*_entry` titles are informational.

## `work_index_overrides.json`

Hand-curated corrections, applied on top of the audit:

- `auditPathAliases`: audit `.txt` path -> page in `raw/`, where the default
  mapping can't find it.
- `extraAuditPages`: pages the audit missed, added to the entry with the given
  `ll_author`.
- `promotedWorkPages`: page-list corrections for works promoted into the
  Library (`works/*.json`), by work id: `pages` / `redundantOrAlternatePages`
  replace the computed lists, `removePages` drops pages.
- `unauditedWorks`: author, title, period (and optionally status and
  `libraryWorkIds`) for pages the audit doesn't cover, keyed by the work's
  first page.

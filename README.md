# Component Code Audit

Audit authored CSS, JavaScript, and attribute hooks across Georgia Power pages and
Experience Fragments. The default **Components** view hides plain HTML and empty
components. “Added” means present in the exported authored content; this is not a
change-history comparison or a live-page runtime audit.

## What is flagged

- **CSS:** style attributes (single-, double-quoted, or unquoted), `<style>` blocks,
  and stylesheet links. Ordinary rich-text styling counts.
- **JavaScript:** executable inline scripts, external scripts, event handlers, and
  `javascript:` URLs. JSON-LD, JSON data, and import maps do not count as executable scripts.
- **Attribute hooks:** nonempty `class` and `id` values and any `data-*` attribute.
  These are review signals, not proof that custom styling or behavior executes.

The shared HTML tokenizer skips comments and raw-text bodies when finding tags and
attributes. Source is displayed as escaped text; scripts are never executed and
referenced assets are never fetched. CSS declarations and selectors are analyzed
locally; linked stylesheets and scripts are identified without following their URLs.

## Reviewing components

The Components tab filters by type, Pages/XF scope, CSS, JavaScript, attribute hooks,
or CSS + JavaScript. Counts are component counts, not declaration counts. Results
are paginated in groups of 50. **Show all components** includes plain and empty
components; other selected filters still apply.

Select a component to see the source property for each finding, highlighted evidence,
full source, author editor and CRXDE links, and flag/note controls. Multivalued
properties retain indexed labels such as `description[0]`.

Style patterns and Selectors retain the CSS categories, risk filters, and instance
drill-down across all imported types. Categories and source lenses default to all
CSS rather than spacing only. XF and GP page tabs remain available for browsing.

## Trends filtering and Excel reports

Open **Trends → Inline styles** to find shared combinations. The first visit starts
at the largest exported combination (currently **12 styles together**) and
**Minimum components = 2**. Choose any size, including 1 for individual declarations,
or use **Next smaller size**. Matching includes ordered, nonadjacent declarations
inside larger style attributes. It never includes CSS inside `<style>` blocks.

**Find maximum**, beside Minimum components, sets the minimum to the highest
component count for the current size and other filters. It checks all results pages
and ignores the previous minimum. If nothing matches, the minimum stays unchanged.
**More filters** contains component type, Pages/XF scope, CSS category and review
notes. Search matches CSS and proposed names. Counts are recalculated within those
filters. Results rank by distinct pages/XFs, then components, then occurrences.

**Style blocks** matches complete blocks separately, retaining existing selectors and
media conditions. **Styles together** and **Next smaller size** are hidden in that
view; switching back restores the chosen inline size. Results are paginated in groups
of 30. The Authoring pages column links to the Southern Company editor; expand it
for additional pages/XFs. Expand a CSS row for exact field evidence, original element
source, review notes and component-viewer navigation. Source is always inert text.

### Export filtered results to Excel

Use **Export filtered results to Excel** beside the results count. The download
includes **every matching result across every results page**, regardless of the page
you are viewing. The table, Find maximum and export use the same filtering and ranking
logic. Saved drafts do not exclude matches. Empty results disable export.

Files are named `gpc-trends-inline-YYYY-MM-DD.xlsx` or
`gpc-trends-style-blocks-YYYY-MM-DD.xlsx`, with the export date in UTC.

The workbook contains **Details** only, with one row per source match: a readable
trend ID, component type, scope, component path, source field, element type, matched
CSS, original source fragment, classes, page/XF path, and clickable Author/CRXDE links.
Element and declaration position columns are omitted.

Trend IDs look like **Inline 0001 — Color + Margin** or **Style block 0001 — Color**.
Numbers follow the filtered results order and identify a trend within that workbook;
all its source matches share that ID. Inline labels describe CSS properties, while
block labels describe CSS categories. Existing internal trend identities stay intact.

Export time and active filters appear above each Details table. The Matching trends
totals row is omitted. Headers are frozen, column filters are enabled, and CSS is wrapped.
Details use each occurrence's
own declarations and comments; Original source fragment retains the authored opening
tag or full style block. Nothing is merged, deduplicated, simplified or executed.
Duplicate declarations, values, case, order, comments, `!important`, quotes and source
line endings remain intact. Authored strings are text cells, never Excel formulas.
Different values for the same property remain optional review notes; shorthand pairs
such as `background` followed by `background-color` are not labeled conflicts.

Long text continues in numbered columns, split at no more than 30,000 characters or
250 line breaks per cell. Join those columns **without separators** to recover the
full text. More than 30,000 source matches create **Details 1**, **Details 2**, etc.,
keeping both links on every row within Excel's per-sheet limits. No results or source
text are silently truncated. Matches
can overlap between combinations; occurrence counts are not additive cleanup savings.

Filters and evidence are captured when export starts. You may change filters while
it runs; the workbook retains the original selection. Progress and failures appear
beside the results. Generation runs locally in a browser worker using the vendored
**ExcelJS 4.4.0** browser bundle, embedded by the build. It uses no runtime CDN and
sends no content to an export server. Larger exports may take several seconds.

### Temporarily hidden draft tools

The stylesheet composer, selection checkboxes, draft sidebar, CSS download, saved
proposal controls and draft progress are temporarily hidden behind the internal
`TREND_DRAFTS_ENABLED` flag in the private template. Their implementation remains for
later reuse. With the flag off, the app does not rewrite saved drafts or legacy
proposals and never applies their coverage to reports, evidence or Find maximum.

Local keys `component-audit-trend-draft-v2` and
`component-audit-trend-proposals-v1` remain intact, including selectors, edited CSS,
source references and stale rules. Flags, notes, credentials and annotation snapshots
are unchanged. Trends use versioned encrypted analysis in `analysis.json`; older or
missing trend data is rebuilt lazily using the same analyzer as the private build.
Large combinatorial searches report limits rather than truncating matches. No AEM
content, repository stylesheet or published app is changed by filtering or exporting.

## Input exports

Place complete QueryBuilder responses in the private build toolchain's `build/raw/`:

| File | Component type | Content field |
|---|---|---|
| `freeform.json` | Freeform | `text` |
| `contentcard.json` | Content card | `text` |
| `herobanner.json` | Hero banner | `description` |
| `textoverimage.json` | Text over image | `text` |
| `thumbnailwithtext.json` | Thumbnail with text | `desc` |

The build reads all `raw/*.json`, infers type from filename, and accepts strings or
arrays of strings in the mapped field. Missing fields remain as empty components.
Only `/content/georgia-power` and `/content/experience-fragments/georgiapower` are
included. The build prints exported, in-scope, and excluded counts per file.

Responses must have `success: true`, `more: false`, and counts matching `hits`.
Unknown filenames, invalid field types, and conflicting duplicate node paths fail
before output is written. Identical duplicate nodes are deduplicated. See
[queries.txt](queries.txt) for query guidance.

## Rebuilding

Requirements: Python 3.9+, `cryptography`, and Node.js 18+ (`node` on PATH, or set
`FF_AUDIT_NODE` to its executable). The same JavaScript analyzer (no npm dependencies) runs
in Node during builds and is embedded into the app for browser fallback.

```sh
cd build
pip install -r requirements.txt
node test_analyzer.cjs
node test_trends.cjs
node test_trend_export.cjs
python3 test_build.py
python3 build.py                 # full app + data rebuild
python3 build.py --data-only     # data refresh after the current app is deployed
```

Optional browser integration checks require Playwright and installed Chrome: serve the
deploy folder on localhost:8765, then run `node build/test_browser.cjs` from the repo
root. Set `PLAYWRIGHT_MODULE` if Playwright is installed outside the normal module path.
Excel checks round-trip source evidence, links, long-text continuations and multiple
detail sheets, including the full dataset and missing-analysis fallback.
The test reads existing credentials into process memory and uses the login form; it
does not serve credentials or write test notes to your normal browser profile.

Use `--input raw/` or `--input raw/herobanner.json` to select a directory or one named
export. A single-file build replaces the dataset with that file's components; use
the directory for the full audit. Add new type-to-property mappings in `build.py`
before importing a new filename.

The build reuses the existing credential salt when credentials still unlock
`index.html`. Do not use `--rotate-salt` unless intentionally rotating it. Credentials
remain in the private build toolchain; never copy that toolchain to public git history.

## Files and annotations

- `index.html`: app shell, shared analyzer and local Excel worker bundle; no authored source.
- `freeforms.json`: encrypted, compressed source keyed by `jcr:path`. The historical
  filename is retained; records now include `componentType`, `fields`, and compatible `text`.
- `analysis.json`: encrypted CSS patterns, selectors, per-component findings, import totals, and versioned trend groups.
- `querybuilder.json`: optional encrypted shared flags, notes, and snapshots, exported
  by the UI. Builds never overwrite this file.
- `build/`: private toolchain, tests, and plaintext raw exports. Never publish publicly.

Flags and notes retain their existing browser storage keys and node identities.
Plain components are retained in the dataset even when hidden, so their annotations
stay live. Removed nodes retain snapshots under Legacy. Old text-only snapshots and
source records remain readable. UI filter state has a new key so the updated app
initially opens Components; saved flags and notes are unaffected.

Flags/notes auto-save in localStorage, which is plaintext and per browser origin.
Export `querybuilder.json` to share an encrypted baseline. The application merges
shared and local annotations by timestamp. Closing the tab clears the session login.

## Preview and publication

Serve the deploy folder with `python3 -m http.server 8000`, then visit localhost:8000.
Opening an HTML file directly offers manual encrypted-data file pickers if fetch fails.

Publish only the app shell, encrypted data, optional encrypted annotations, and public
docs. `build/publish.py` remains a private-repository mirror tool: it includes every
`build/raw/*.json` unless `--no-raw` is passed, and refuses to target its own source
folder. Use `--dry-run` to inspect the mirror before publishing. It does not rebuild.

Data and shared annotations use AES-256-GCM, with a key derived from credentials via
PBKDF2. Anyone with those credentials can read the full dataset. Browser-local flags
and snapshots are not encrypted. Keep raw exports and credentials private.

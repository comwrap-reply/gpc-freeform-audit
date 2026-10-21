# Component Code Audit

Audit authored CSS, JavaScript, and attribute hooks across Southern Company sites.
Use the site dropdown to switch among the configured content roots; every view and
report is scoped to that selected site. The default **Components** view hides plain
HTML and empty components. “Added” means present in the exported authored content;
this is not a change-history comparison or a live-page runtime audit.

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
CSS rather than spacing only. XF and Pages tabs remain available for browsing.

Component labels use the same colors across sites and views: Freeform is teal,
Content card blue, Hero banner violet, Text over image amber, and Thumbnail with
text pink. Labels always include the component name.

In **Pages**, use **Component** to select a type. The page list shows pages with
that type, with counts and color labels; the source viewer shows only components
of that type. Search, styled, spacing, and flagged/noted filters combine with the
type filter, requiring a component to meet the selected conditions. The selection
is saved separately for each site. **All types** restores the complete view, and
source links clear incompatible filters so their target remains visible.

## Third-party services

Open **Third-party services** to identify services such as YouTube, Vimeo, Formstack,
Brightcove, Google Maps, and social widgets in the selected site's exported content.
Providers are grouped by purpose, with distinct component and page/XF counts. The tab
includes all five component types, including iframe-only components that have no
CSS, JavaScript, or attribute-hook review signals.

**Embeds & integrations** is the default view. **Ordinary links** and **Possible
JavaScript references** are separate views; a link or a URL string does not establish
that a service executes. Use search, purpose, component type, and Pages/XF filters to
narrow the results. Expand a provider for exact source evidence, source-field labels,
and links to the component viewer, Author, and CRXDE. Filter state is saved per site.

Provider rules use parsed hostnames and explicit domain/endpoint matches. Unmatched
hostnames appear under **Unclassified hosts**, which may contain internal and external
hosts. Comments and plain-text brand mentions are ignored. The scanner reads source
without executing JavaScript or loading embeds; it cannot establish runtime behavior
or detect URLs assembled dynamically by scripts. Older analysis files are recomputed
with the same scanner in the browser. Raw source and findings remain encrypted in
the deployed data files.

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

Files are named `<site-id>-trends-inline-YYYY-MM-DD.xlsx` or
`<site-id>-trends-style-blocks-YYYY-MM-DD.xlsx`, with the export date in UTC.

The workbook contains **Details** only, with one row per source match: a readable
trend ID, component type, scope, component path, source field, element type, matched
CSS, original source fragment, classes, page/XF path, and clickable Author/CRXDE links.
Element and declaration position columns are omitted.

Trend IDs are **Trend 1**, **Trend 2**, **Trend 3**, and so on for both inline styles
and style blocks. Numbers follow the filtered results order and identify a trend
within that workbook; all its source matches share that ID. Existing internal trend
identities stay intact.

Export time and active filters appear above each Details table. The Matching trends
totals row and Reading this workbook row are omitted. Headers are frozen, column filters are enabled, and CSS is wrapped.
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

The private updaters store complete QueryBuilder responses under
`build/raw/<site-id>/` for website pages and `build/raw/<site-id>/xf/` for Experience
Fragments:

| File | Component type | Content field |
|---|---|---|
| `freeform.json` | Freeform | `text` |
| `contentcard.json` | Content card | `text` |
| `herobanner.json` | Hero banner | `description` |
| `textoverimage.json` | Text over image | `text` |
| `thumbnailwithtext.json` | Thumbnail with text | `desc` |

The build recursively reads `raw/**/*.json`, infers type from filename, and accepts
strings or arrays of strings in the mapped field. Missing fields remain as empty
components. The containing directories identify both site and source scope. Page
exports must remain under the site's website content path; XF exports must remain
under that site's explicitly approved Experience Fragment roots. A response from any
other website or fragment tree is rejected. Legacy flat Georgia Power exports remain
supported until a `raw/georgia-power/` refresh replaces them.

Responses must have `success: true`, `more: false`, and counts matching `hits`.
Unknown filenames, invalid field types, and conflicting duplicate node paths fail
before output is written. Identical duplicate nodes are deduplicated. See
[queries.txt](queries.txt) for query guidance.

## Rebuilding

Requirements: Python 3.9+, `cryptography`, and Node.js 18+ (`node` on PATH, or set
`FF_AUDIT_NODE` to its executable). The same JavaScript analyzer (no npm dependencies) runs
in Node during builds and is embedded into the app for browser fallback.

Create the private build environment once. The updater automatically uses this
environment for rebuilding even when it is launched with `python3`:

```sh
cd build
python3 -m venv .venv
.venv/bin/python -m pip install -r requirements.txt
```

```sh
cd build
node test_analyzer.cjs
node test_services.cjs
node test_trends.cjs
node test_trend_export.cjs
python3 test_build.py
python3 test_update_site.py
python3 test_update_experience_fragments.py
python3 build.py                 # full app + data rebuild
python3 build.py --data-only     # data refresh after the current app is deployed
```

To refresh one site from `https://author.southerncompany.com`, run the site updater.
It fetches all five component types with a 5-second pause between requests, validates
the complete batch before replacing that site's raw files, rebuilds the encrypted
data, and syncs it to the sibling public checkout:

```sh
python3 update_site.py georgia-power --cookie-file /path/to/cookies.txt
python3 update_site.py                         # interactive site menu
python3 update_site.py alabama-power --print-urls
```

`cookies.txt` must use Netscape cookie-file format and come from an authenticated
author session. As an alternative, set `AEM_COOKIE` to the browser's Cookie request
header. Existing `PROD_USERNAME` and `PROD_PASSWORD` environment variables are also
accepted for HTTP Basic authentication, so no authentication arguments are needed
when those are already exported. See [queries.txt](queries.txt) for every site id and
authentication option.

To refresh only the Experience Fragments assigned to a site, use the XF updater. It
uses the same authentication, five component mappings, 5-second delay, rebuild, and
public-data sync. Results are stored separately under `raw/<site-id>/xf/`:

```sh
python3 update_experience_fragments.py georgia-power
python3 update_experience_fragments.py agl
python3 update_experience_fragments.py alabama-power --print-urls
python3 update_experience_fragments.py --all
```

The XF paths come only from `sites.json`, which was mapped from the supplied
`/content/experience-fragments` tree. The updater validates every returned `jcr:path`
against the selected site's roots before replacing any files. `--all` runs all 70
site/component queries sequentially, waits 5 seconds between every query, and performs
one encrypted rebuild and public-data sync after all 14 sites have finished.

Optional browser integration checks require Playwright and installed Chrome: serve the
deploy folder on localhost:8765, then run `node build/test_browser.cjs` from the repo
root. Set `PLAYWRIGHT_MODULE` if Playwright is installed outside the normal module path.
Excel checks round-trip source evidence, links, long-text continuations and multiple
detail sheets, including the full dataset and missing-analysis fallback.
The test reads existing credentials into process memory and uses the login form; it
does not serve credentials or write test notes to your normal browser profile.
`node build/test_services_browser.cjs` runs focused service-tab checks against the
same local preview, including filters, site isolation, escaped evidence and older
analysis fallback.

Use `--input raw/` for the complete audit or point at one export while developing.
Add new type-to-property mappings in `build.py`, `update_site.py`, and
`update_experience_fragments.py` before importing a new filename.

The build reuses the existing credential salt when credentials still unlock
`index.html`. Do not use `--rotate-salt` unless intentionally rotating it. Credentials
remain in the private build toolchain; never copy that toolchain to public git history.

## Files and annotations

- `index.html`: app shell, shared analyzer and local Excel worker bundle; no authored source.
- `freeforms.json`: encrypted, compressed source keyed by `jcr:path`. The historical
  filename is retained; records now include `componentType`, `fields`, and compatible `text`.
- `analysis.json`: encrypted per-site CSS patterns, selectors, per-component findings, import totals, and versioned trend groups.
- `querybuilder.json`: optional encrypted shared flags, notes, and snapshots, exported
  by the UI. Builds never overwrite this file.
- `build/`: private toolchain, site catalog, updater, tests, and plaintext raw exports. Never publish publicly.

Flags and notes retain their existing browser storage keys and node identities.
Plain components are retained in the dataset even when hidden, so their annotations
stay live. Removed nodes retain snapshots under Legacy for their own site. Old
text-only snapshots and source records remain readable. UI filter state is saved per
site; saved flags and notes remain globally path-keyed and are unaffected.

Flags/notes auto-save in localStorage, which is plaintext and per browser origin.
Export `querybuilder.json` to share an encrypted baseline. The application merges
shared and local annotations by timestamp. Closing the tab clears the session login.

## Preview and publication

Serve the deploy folder with `python3 -m http.server 8000`, then visit localhost:8000.
Opening an HTML file directly offers manual encrypted-data file pickers if fetch fails.

Publish only the app shell, encrypted data, optional encrypted annotations, and public
docs. `build/publish.py` remains a private-repository mirror tool: it includes every
`build/raw/**/*.json` unless `--no-raw` is passed, and refuses to target its own source
folder. Use `--dry-run` to inspect the mirror before publishing. It does not rebuild.

Data and shared annotations use AES-256-GCM, with a key derived from credentials via
PBKDF2. Anyone with those credentials can read the full dataset. Browser-local flags
and snapshots are not encrypted. Keep raw exports and credentials private.

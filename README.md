# Freeform Style Audit

An interactive, credential-gated audit of every `/apps/global/components/content/freeform`
instance across Georgia Power pages (`/content/georgia-power`) and Georgia Power
Experience Fragments (`/content/experience-fragments/georgiapower`), focused on inline
styling — spacing overrides (padding / margin / gap) plus every other CSS category
(layout, sizing, typography, color, effects), with migration-risk tagging.

The site is a small static app plus two encrypted data files. All fragment source, page
content, and author paths are **AES-256-GCM encrypted**; the decryption key is derived
from the login credentials (PBKDF2, 310k iterations), so the files are safe to host on a
public GitHub Pages URL. Credentials are distributed out-of-band — never commit them to
this repo, this README, or any issue/PR.

Everything is keyed by `jcr:path`, which is unique and stable per freeform. That is the
whole design: you can re-run the AEM query and replace the data at will without breaking
your flags and notes, and without rebuilding `index.html`.

## What's in the tool

- **Style patterns** — every inline CSS declaration found across **both** XF freeforms
  and GP page freeforms, ranked by how many freeforms share it, with drill-down to each
  instance (author editor link, CRXDE node link, highlighted context snippet). Filter by
  **scope** (All / XF / Pages), by **category** (spacing, layout, sizing, type, color,
  effect, other — spacing is preselected), and by **risk** (`!important`, negative values,
  hardcoded px, hex colors, `position:absolute/fixed`, `@media`). Expanded rows show XF
  instances first, then page freeforms grouped by page and capped with "show more".
- **Selectors** — CSS selectors carrying any rule inside `<style>` blocks, across both
  scopes (Core Component overrides like `li.cmp-tabs__tab` surface here). Same scope,
  category, and risk filters as patterns.
- **XF fragments** — browsable source viewer for all GP-brand XF freeforms with style
  attributes and style blocks highlighted, each declaration coloured by category; the
  **category lens** dims everything except the chosen category (`!important` and negative
  values are underlined in place).
- **GP pages** — all Georgia Power pages containing freeforms; click a page to see every
  freeform on it with the same viewer, links, and lens.
- **Flagged & notes** — flag any freeform and attach notes from anywhere in the tool;
  build a working set for remediation / EDS migration review.
- **Legacy** — notes and flagged code for freeforms that are no longer in the export.
  Annotating a freeform snapshots its source, so when it is later deleted, moved, or
  renamed in AEM the note keeps the code it was about. Nothing here is ever removed
  automatically.

## Repo layout

| Path | Committed? | What it is |
|---|---|---|
| `index.html` | ✅ push | The app shell plus a small credential sentinel. No content, ~85 KB. Only changes when the template or the credentials change. |
| `freeforms.json` | ✅ push | Freeform source keyed by `jcr:path`, encrypted + gzipped (~1.1 MB). Replace this to refresh the data. |
| `analysis.json` | ✅ push | Derived analysis (patterns, selectors, per-freeform category/risk counts) over all scopes and properties. Paths are interned into one table and rows reference them by index, so the file stays small; regenerated alongside `freeforms.json`. |
| `querybuilder.json` | ✅ push (optional) | Encrypted shared baseline of flags, notes, and source snapshots, keyed by `jcr:path`. Written by the Export button in the tool — never by the build. |
| `queries.txt` | ✅ push | The AEM QueryBuilder request that produces the raw data, with how to run it and why each parameter is there. |
| `README.md` / `.gitignore` | ✅ push | This file, and the rule that keeps build inputs out of the repo. |
| `build/` | ❌ keep local | Everything needed to rebuild, including how-to-run docs at the top of `build.py`. **Never push** — `raw/freeform.json` is the QueryBuilder export in plaintext, `template.html` is the unencrypted app shell, and `build.py` holds the login. |

## Deploying

1. Push `index.html`, `freeforms.json`, `analysis.json` (plus this README / .gitignore).
2. Repo → Settings → Pages → deploy from branch → `main`, root (`/`).
3. The site serves at `https://<user>.github.io/<repo>/`.

> GitHub Pages sites are publicly reachable at their URL even when the repo is private
> (private-repo Pages requires a paid plan; on free plans use a public repo). Either way,
> the URL being public is fine — the content is ciphertext without the credentials. A
> private repo additionally protects the encrypted files at the source and is preferred.

## Refreshing the data

1. Re-run the QueryBuilder export on author and save it over `build/raw/freeform.json`.
   The query, how to run it, and what every parameter is for live in
   [`queries.txt`](queries.txt) — that file is the canonical copy, so it does not drift
   from this README.

2. Rebuild the data only — `index.html` is not touched, so the deployed app and its
   PBKDF2 salt stay exactly as they are:

   ```bash
   cd build
   pip install -r requirements.txt      # once
   python3 build.py --data-only
   ```

   No credentials on the command line: they live in the `CREDENTIALS` block at the top
   of `build.py`, which also documents how to change them. `--user` / `--password` (or
   `FF_AUDIT_USER` / `FF_AUDIT_PASSWORD`) override them for a single run. The build
   prints which source it used, so you can confirm before pushing.

3. Commit the two regenerated JSON files. Your `querybuilder.json` keeps working: it is
   keyed by `jcr:path`, so flags and notes reattach to the same freeforms, and anything
   that dropped out of the new export moves into the Legacy section with the snapshot
   taken when you annotated it.

Drop `--data-only` when you have also changed `template.html` and want a new `index.html`.
That reuses the existing salt as long as the credentials still unlock the current
`index.html`, so an exported `querybuilder.json` keeps decrypting. Pass `--rotate-salt`
only when you intend to invalidate it.

The build prints which of those two happened; the last line is either
`salt reused from index.html` or `NEW salt minted`.

## Flags, notes & the Legacy section

- Flags and notes auto-save to the **browser's localStorage** on every change (plaintext,
  local to your machine, per-origin — the Pages URL and a local file: URL keep separate state).
- Annotating a freeform also snapshots its full source, refreshed on load while the
  freeform is still in the export. That snapshot is what the Legacy section renders once
  the path disappears from `freeforms.json`.
- To publish a shared baseline: **Flagged & notes → Export querybuilder.json**, then commit
  the downloaded file to the repo root. On load, the tool fetches it, decrypts it with the
  session key, and merges with local changes. Merging is per field: the newer timestamp
  wins for the flag and the note, and a snapshot is never dropped just because the other
  side lacks one.
- A `flags.json` from an older build is still read on load and on import, so an existing
  committed baseline migrates by itself. You can delete it once you have exported
  `querybuilder.json`.
- Nothing is pruned automatically. Each legacy entry has its own **Delete**, and the
  section header has **Delete all legacy** for clearing a batch once you are done with
  them; both confirm first, and both point out that the snapshot is the last copy of
  source that is no longer in AEM. Live flags and notes are never touched by either. If a
  path reappears in a later export the entry leaves the Legacy section on its own.
- If the browser runs out of storage, the tool sheds snapshots it can rebuild from
  `freeforms.json` first and tells you; an orphan's snapshot is the last copy and is
  never shed.

## Running it locally

The data files are fetched at runtime, and browsers block `fetch` between `file://`
pages, so serve the folder instead:

```bash
cd <repo root> && python3 -m http.server 8000   # then open http://localhost:8000/
```

Opening `index.html` straight off disk still works: the unlock screen notices the fetch
failed and offers a file picker for `freeforms.json` and `analysis.json`.

## Rotating credentials

Edit the `CREDENTIALS` block at the top of `build/build.py`, run a full build (no
`--data-only`), and push the new `index.html` plus the two data files. Old credentials stop
working immediately (they can no longer derive the key).
A new salt is minted in this case, so a `querybuilder.json` exported under the old
credentials will not decrypt — your localStorage copy is plaintext and merges
automatically, so open the new build once, confirm the Flagged and Legacy sections look
right, then re-export and commit.

## Security model (honest version)

- The data files and `querybuilder.json` are real ciphertext — there is nothing readable
  in the repo or on the wire without the credentials.
- `querybuilder.json` contains freeform source for every path you annotated, so treat it
  as exactly as sensitive as the data files. It stays encrypted for that reason.
- Anyone with the URL **and** the credentials has everything; treat the password like
  the content itself.
- Because the ciphertext is public, offline brute-force against a weak password is
  possible in principle. The current password is adequate for keeping the site out of
  crawlers and casual visitors, not for high-sensitivity data. Rotate to a longer
  passphrase if the audience for this repo widens.
- Login persists per browser tab (sessionStorage); closing the tab locks the tool.
  UI state (tab, filters, selections) persists across reloads via localStorage.

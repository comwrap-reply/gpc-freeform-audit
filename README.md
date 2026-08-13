# Freeform Style Audit

An interactive, credential-gated audit of every `/apps/global/components/content/freeform`
instance across Georgia Power pages (`/content/georgia-power`) and Georgia Power
Experience Fragments (`/content/experience-fragments/georgiapower`), focused on inline
styling and spacing overrides (padding / margin / gap).

The tool is a single static HTML file. All fragment source, page content, and author
paths are embedded **AES-256-GCM encrypted**; the decryption key is derived from the
login credentials (PBKDF2, 310k iterations), so the file is safe to host on a public
GitHub Pages URL. Credentials are distributed out-of-band — never commit them to this
repo, this README, or any issue/PR.

## What's in the tool

- **Spacing patterns** — every padding/margin/gap declaration found in XF freeforms,
  ranked by how many fragments and fragment families share it, with drill-down to each
  instance (author editor link, CRXDE node link, highlighted context snippet).
- **Selectors** — CSS selectors carrying spacing rules inside `<style>` blocks
  (Core Component overrides like `li.cmp-tabs__tab` surface here).
- **XF fragments** — browsable source viewer for all GP-brand XF freeforms with style
  attributes, style blocks, and spacing declarations highlighted; "spacing lens" toggle
  dims everything except spacing.
- **GP pages** — all Georgia Power pages containing freeforms; click a page to see every
  freeform on it with the same viewer, links, and lens.
- **Flagged & notes** — flag any freeform and attach notes from anywhere in the tool;
  build a working set for remediation / EDS migration review.

## Repo layout

| Path | Committed? | What it is |
|---|---|---|
| `index.html` | ✅ push | The built, encrypted tool. This is the whole deployed site. |
| `flags.json` | ✅ push (optional) | Encrypted shared baseline of flags + notes. Created by the Export button in the tool. |
| `README.md` | ✅ push | This file. |
| `.gitignore` | ✅ push | Keeps the build inputs out of the repo. |
| `build/` | ❌ keep local | Everything needed to rebuild `index.html`. **Never push** — `querybuilder.json` contains all freeform source in plaintext, and `template.html` is the unencrypted app shell. |

## Deploying

1. Push `index.html` (plus this README / .gitignore) to the repo.
2. Repo → Settings → Pages → deploy from branch → `main`, root (`/`).
3. The site serves at `https://<user>.github.io/<repo>/`.

> GitHub Pages sites are publicly reachable at their URL even when the repo is private
> (private-repo Pages requires a paid plan; on free plans use a public repo). Either way,
> the URL being public is fine — the content is ciphertext without the credentials. A
> private repo additionally protects the encrypted files at the source and is preferred.

## Flags & notes workflow

- Flags and notes auto-save to the **browser's localStorage** on every change (plaintext,
  local to your machine, per-origin — the Pages URL and a local file: URL keep separate state).
- To publish a shared baseline: **Flagged & notes → Export flags.json**, then commit the
  downloaded file to the repo root next to `index.html`. On load, the tool fetches it,
  decrypts it with the session key, and merges with local changes (newer timestamp wins
  per item, including un-flags).
- The exported file is encrypted with the same credentials as the page — safe to commit.
  Legacy plaintext flags.json files are still accepted on load/import for migration.

## Rebuilding `index.html` (local only)

When content changes or you want fresh data:

1. Re-run the QueryBuilder export on author and save it over `build/querybuilder.json`:

   ```
   /bin/querybuilder.json?type=nt:unstructured&property=sling:resourceType&property.value=global/components/content/freeform&group.p.or=true&group.1_path=/content/georgia-power&group.2_path=/content/experience-fragments&p.limit=-1&p.hits=selective&p.properties=jcr:path text
   ```

   (The build filters XFs to the `georgiapower` brand itself, so the broader XF path in
   the query is fine.)

2. Build:

   ```bash
   cd build
   pip install -r requirements.txt
   python build.py --user <username> --password '<password>'
   ```

   This writes `../index.html`. Commit and push it.

3. **Flags caveat after a rebuild:** every build generates a fresh PBKDF2 salt, so a
   `flags.json` exported from the *previous* build will not decrypt in the new one.
   Your localStorage flags are unaffected and merge automatically — open the new build,
   confirm the Flagged tab looks right, re-export, and commit the new `flags.json`.

## Rotating credentials

Rebuild with the new `--user` / `--password` and push the new `index.html`. Old
credentials stop working immediately (they can no longer derive the key). Then follow
the flags caveat above to re-export `flags.json` under the new credentials.

## Security model (honest version)

- The payload and flags.json are real ciphertext — there is nothing readable in the
  repo or on the wire without the credentials.
- Anyone with the URL **and** the credentials has everything; treat the password like
  the content itself.
- Because the ciphertext is public, offline brute-force against a weak password is
  possible in principle. The current password is adequate for keeping the site out of
  crawlers and casual visitors, not for high-sensitivity data. Rotate to a longer
  passphrase if the audience for this repo widens.
- Login persists per browser tab (sessionStorage); closing the tab locks the tool.
  UI state (tab, filters, selections) persists across reloads via localStorage.

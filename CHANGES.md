# Change Log

## 2026-05-28 — Fix Hugo server startup

### Problem
`hugo server` was not starting. The site uses Hugo Modules (`go.mod`) to pull in the HugoBlox theme, which requires Go to be installed. Go was missing and no `go.sum` file existed, so Hugo could not resolve any theme dependencies.

After installing Go and fetching modules, three additional Hugo compatibility errors blocked the build (Hugo v0.162.1 is significantly newer than the theme was authored for).

### Changes

**1. Installed Go via Homebrew**
- `brew install go` (Go v1.26.3)
- Required by Hugo Modules to fetch and manage theme dependencies.

**2. Updated theme modules to latest versions**
- Ran `hugo mod get -u` then `hugo mod tidy`
- Generated `go.sum` (was missing)
- `blox-bootstrap/v5`: v5.9.6 → v5.9.7
- `blox-plugin-netlify`: v1.1.2-… → v1.2.0
- `blox-plugin-reveal`: v1.1.2 → v1.2.4
- `blox-seo`: v0.2.2 → v0.3.1
- `blox-core`: v0.3.1 → v0.4.1

**3. Created `layouts/shortcodes/table.html`** (new file)
- Local override of the theme's `table.html` shortcode.
- The theme used `getCSV`, which was removed in Hugo v0.141.0.
- Replaced with `transform.Unmarshal` (the current API) for both local and remote CSV sources.

**4. Fixed `content/authors/_index.md`**
- Renamed `_build:` → `build:` in front matter (and in the `cascade` block).
- The `_build` key was removed in Hugo v0.145.0.

**5. Fixed `content/publication/odonga-2025-evidence/index.md`**
- Changed `publishDate: '2025-01-29T18:43:01 UTC'` → `'2025-01-29T18:43:01Z'`
- The `UTC` suffix is not a valid RFC3339 date format; Hugo rejected it.

**6. Created `layouts/partials/blocks/about.biography.html`** (new file)
- Local copy of the same file from the module cache.
- Hugo was unable to resolve it through the module mount during builds, causing a fatal error.
- Local copy ensures it is always found.

### Result
`hugo server` now builds and serves the site cleanly at `http://localhost:1313/`.
Remaining output is deprecation `WARN`s emitted by the theme itself (non-fatal).

# Taxonomy Browser & Term Wizard

A single, self-contained `index.html` — no build, no server, no backend.

- **Browse** existing AAIF taxonomy terms with search + filters.
- **Propose / edit** a term, then **download a `create-pr.sh`** script that
  makes the branch, surgical commit, push, and PR for you.

## Data source (no CORS)

The source of truth is
[`taxonomy/taxonomy-data.js`](https://github.com/MatthewKhouzam/ws-taxonomy-landscape/blob/main/taxonomy/taxonomy-data.js),
which assigns to `window.AAIF_TAXONOMY = [...]`.

Because it's a script that assigns to a global, we load it via `fetch()` at
this **raw** URL and evaluate the assignment in a sandboxed scope:

```
https://raw.githubusercontent.com/MatthewKhouzam/ws-taxonomy-landscape/main/taxonomy/taxonomy-data.js
```

We deliberately do **not** use a `<script src>` tag. `raw.githubusercontent.com`
serves the file as `Content-Type: text/plain` with
`X-Content-Type-Options: nosniff`, so modern browsers **refuse to execute it as
a script** (MIME mismatch under nosniff) and, worse, fire neither `onload` nor
`onerror` reliably. `fetch()` avoids all of that, and the response sends
`access-control-allow-origin: *`, so there are **no CORS issues**.

Do **not** use the `github.com/.../blob/...` URL — that returns an HTML page,
not JS. To freeze a snapshot, replace `main` with a commit SHA.

## Why a downloadable script instead of a backend

A static page can't run git. So the wizard generates a fully self-contained
`create-pr.sh` (bash + an embedded Python surgical editor). You run it from the
repo root:

```bash
bash create-pr.sh
```

It refuses if the data file is dirty, branches off `main`, applies a
comment-preserving edit to only `taxonomy/taxonomy-data.js`, commits with
`-s` (Signed-off-by), pushes, and opens the PR via `gh` (or prints the compare
URL if `gh` isn't installed).

## Run

Just open `index.html` in a browser, or serve the folder:

```bash
python3 -m http.server -d taxonomy-browser 8000
# http://localhost:8000
```

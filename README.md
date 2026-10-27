# csaf-files

CSAF and VEX documents published by the Secvisogram team.

Every document in `advisories/` is validated, signed and published to
`https://secvisogram.github.io/.well-known/csaf/` by a GitHub Actions workflow. Adding a
document to `main` is all it takes; there is no manual upload step and no hosting cost.

Publishing happens in two repositories: this repo detects the change and triggers a
workflow in [`secvisogram/secvisogram.github.io`](https://github.com/secvisogram/secvisogram.github.io),
which performs the actual validation, signing and publishing. See
[Architecture](#architecture) below.

## Adding a VEX document

Put the JSON file in `advisories/`, using one subdirectory per year:

```
advisories/<year>/<tracking-id>.json
```

- `<year>` is the year of `document.tracking.initial_release_date`
- `<tracking-id>` is `document.tracking.id`, lowercased so the name is safe on every
  file system

For example, the advisory with the tracking ID `csaf-validator-lib-2026-0001` and an
initial release date in 2026 lives here:

```
advisories/2026/csaf-validator-lib-2026-0001.json
```

The workflow reads `advisories/` recursively and derives the published location from the
document itself, so the year directories are a convention that keeps the repository
readable — they do not affect the output.

## Architecture

Publishing is split across two repositories:

1. This repository (`csaf-files`) only holds the source documents. A push to `main`
   touching `advisories/**` triggers the workflow
   [`trigger-publish.yml`](.github/workflows/trigger-publish.yml). It obtains a short-lived
   GitHub App token and sends a `repository_dispatch` event (`csaf-publish`) to
   `secvisogram/secvisogram.github.io`.
2. [`secvisogram/secvisogram.github.io`](https://github.com/secvisogram/secvisogram.github.io)
   receives the dispatch, checks out this repository's `advisories/` folder, validates and
   signs the documents with `csaf-tools/csaf-action`, and commits the result to its own
   `gh-pages` branch under `.well-known/csaf/`.

This repository no longer publishes documents itself; it has no `gh-pages` branch of its
own and no local GitHub Pages site. The published documents are served from
`https://secvisogram.github.io/`, the Pages site of the `secvisogram.github.io` repository
(which also hosts the Secvisogram editor).

## Publishing a document

1. Create a branch and add or update the file under `advisories/`.
2. Open a pull request against `main`.
3. Merge it.

The merge triggers [`trigger-publish.yml`](.github/workflows/trigger-publish.yml), which
dispatches the publish job to `secvisogram/secvisogram.github.io` (see
[Architecture](#architecture)). Follow the run of `publish-csaf.yml` in the **Actions** tab
of that repository. If validation fails, `gh-pages` is left untouched and the log names the
offending document.

## Accessing a published document

Published documents are available under the CSAF well-known path:

```
https://secvisogram.github.io/.well-known/csaf/<tlp>/<year>/<tracking-id>.json
```

`<tlp>` is the lowercased TLP label from `document.distribution.tlp.label`, so a
TLP:WHITE document ends up under `white/`. For example, a document with tracking ID
`example-2026-0001` and `TLP:WHITE` published in 2026 would be available at:

```
https://secvisogram.github.io/.well-known/csaf/white/2026/example-2026-0001.json
```

Each document is accompanied by `.asc`, `.sha256` and `.sha512` files for signature and
checksum verification.

Tools that consume CSAF automatically start at the provider metadata:

```
https://secvisogram.github.io/.well-known/csaf/provider-metadata.json
```

The directories also contain generated `index.html` files, so the tree can be browsed
directly in a web browser.

## References

- [CSAF 2.0 specification](https://docs.oasis-open.org/csaf/csaf/v2.0/csaf-v2.0.html)
- [`csaf-tools/csaf-action`](https://github.com/csaf-tools/csaf-action)
- [Secvisogram](https://github.com/secvisogram/secvisogram) — editor for CSAF documents
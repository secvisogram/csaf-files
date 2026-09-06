# csaf-files

CSAF and VEX documents published by the Secvisogram team.

Every document in `advisories/` is validated, signed and published to GitHub Pages by a
GitHub Actions workflow. Adding a document to `main` is all it takes; there is no manual
upload step and no hosting cost.

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

## Configuring GitHub Pages

This has to be done once per repository, after the first workflow run has created the
`gh-pages` branch:

1. Open **Settings → Pages**.
2. Under **Build and deployment**, set **Source** to `Deploy from a branch`.
3. Select branch `gh-pages` and folder `/ (root)`, then click **Save**.

GitHub then serves the branch at `https://<owner>.github.io/<repository>/`.

## Publishing a document

1. Create a branch and add or update the file under `advisories/`.
2. Open a pull request against `main`.
3. Merge it.

The merge triggers [`publish-csaf.yml`](.github/workflows/publish-csaf.yml), which runs
[`csaf-tools/csaf-action`](https://github.com/csaf-tools/csaf-action). The action
validates the documents, builds the CSAF provider structure, signs the documents and
commits the result to the `gh-pages` branch.

Follow the run in the **Actions** tab. If validation fails, the branch is left untouched
and the log names the offending document. The workflow can also be started manually via
**Actions → Publish CSAF Advisories → Run workflow**.

## Accessing a published document

Published documents are available under the CSAF well-known path:

```
https://<owner>.github.io/<repository>/.well-known/csaf/<tlp>/<year>/<tracking-id>.json
```

`<tlp>` is the lowercased TLP label from `document.distribution.tlp.label`, so a
TLP:WHITE document ends up under `white/`. For the example above:

```
https://secvisogram.github.io/csaf-files/.well-known/csaf/white/2026/csaf-validator-lib-2026-0001.json
```

Each document is accompanied by `.asc`, `.sha256` and `.sha512` files for signature and
checksum verification.

Tools that consume CSAF automatically start at the provider metadata:

```
https://secvisogram.github.io/csaf-files/.well-known/csaf/provider-metadata.json
```

The directories also contain generated `index.html` files, so the tree can be browsed
directly in a web browser.

## References

- [CSAF 2.0 specification](https://docs.oasis-open.org/csaf/csaf/v2.0/csaf-v2.0.html)
- [`csaf-tools/csaf-action`](https://github.com/csaf-tools/csaf-action)
- [Secvisogram](https://github.com/secvisogram/secvisogram) — editor for CSAF documents

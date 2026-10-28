# csaf-files
CSAF files for Secvisogram

## Creating a new advisory

1. Copy `templates/advisory-template.json` to `advisories/<year>/<repo>-<year>-<NNNN>.json`
   (the template lives outside `advisories/` so it is never published).
   `<repo>` is the repository the advisory is about, e.g. `secvisogram`, `csaf-validator-lib` or `csaf-validator-service`.
2. Replace all `{{PLACEHOLDER}}` tokens in the template with real values (see table below) - ideally by opening the file in Secvisogram.
3. For "not affected" (VEX) advisories, set `document.category` to `csaf_vex` and use
   `product_status.known_not_affected` with `flags`/`threats` instead of `known_affected`/`remediations`.
4. Push to `main`; the workflow validates, signs and publishes the document.

### Placeholders

| Placeholder | Fields | Description |
|---|---|---|
| `{{REPOSITORY_NAME}}` | `document.publisher.name`/`.namespace`, `document.references[0].summary`, `product_tree` product name, CPE, PURL | Name of the affected repository, e.g. `secvisogram`, `csaf-validator-lib` or `csaf-validator-service` |
| `{{ADVISORY_ID}}` | `document.tracking.id`, `document.references[0].url` | Full advisory ID, identical to the file name without `.json`, e.g. `csaf-validator-lib-2026-0002` |
| `{{PUBLICATION_YEAR}}` | `document.references[0].url` | Year used in the `advisories/<year>/` path and the publication URL |
| `{{ADVISORY_TITLE}}` | `document.title` | Short title of the advisory |
| `{{ADVISORY_SUMMARY}}` | `document.notes[category=summary].text` | Short summary of the advisory |
| `{{INITIAL_RELEASE_DATE}}` | `document.tracking.initial_release_date`, `revision_history[0].date` | Date/time of the first publication |
| `{{CURRENT_RELEASE_DATE}}` | `document.tracking.current_release_date` | Date/time of the current revision |
| `{{GENERATOR_DATE}}` | `document.tracking.generator.date` | Date/time the document was generated |
| `{{PRODUCT_VERSION}}` | `product_tree` product version, product name, CPE, PURL | Affected version, e.g. `0.1.0` |
| `{{PRODUCT_ID}}` | `product_tree` `product_id`, `product_status.known_affected`, `remediations[].product_ids` | Internal product ID, e.g. `CSAFPID-00001` |
| `{{CVE_ID}}` | `vulnerabilities[].cve` | CVE identifier, e.g. `CVE-2026-12345` |
| `{{DISCOVERY_DATE}}` | `vulnerabilities[].discovery_date` | Date/time the vulnerability was discovered |
| `{{VULNERABILITY_DESCRIPTION}}` | `vulnerabilities[].notes[category=description].text` | Description of the vulnerability |
| `{{REMEDIATION_DETAILS}}` | `vulnerabilities[].remediations[].details` | Description of the fix/workaround |

The outer `product_tree` vendor branch (`"name": "secvisogram"`) is **not** a placeholder - it identifies the GitHub organization and stays the same for every repository.

### Notes

- `{{ADVISORY_ID}}` must be unique: use the next free `NNNN` of the year for that repository (e.g. `csaf-validator-lib-2026-0002`) and keep it identical to the file name (without `.json`), `document.tracking.id` and the `self` reference (`https://secvisogram.github.io/.well-known/csaf/white/{{PUBLICATION_YEAR}}/{{ADVISORY_ID}}.json`).
- If no fix is available yet, change the remediation category from `vendor_fix` to `none_available` or `workaround` and describe the situation in `{{REMEDIATION_DETAILS}}`.
- `{{PRODUCT_ID}}` must be valid and match the described product (CPE, PURL). The same `{{PRODUCT_ID}}` value must be used consistently in `product_tree`, `product_status` and `remediations[].product_ids`.

# cobdfamily/.github

Org-level shared GitHub configuration for the url2code fleet.

## Reusable workflows

- [`reusable-cve-scan.yml`](.github/workflows/reusable-cve-scan.yml)
  — daily Grype scan of a published image's oras-attached
  CycloneDX SBOM, SARIF to the Security tab. Callers supply only
  the `image:` input (e.g. `cobdfamily/brl`) and the schedule/
  dispatch trigger.

A consumer repo calls it like:

```yaml
# .github/workflows/cve-scan.yml
name: cve-scan
on:
  schedule:
    - cron: "40 11 * * *"
  workflow_dispatch:
    inputs:
      tag: { description: "Image tag to scan", required: false, default: "latest" }
jobs:
  scan:
    uses: cobdfamily/.github/.github/workflows/reusable-cve-scan.yml@main
    with:
      image: cobdfamily/brl
      tag: ${{ inputs.tag || 'latest' }}
```

This replaces the per-repo copy-pasted scan logic so a change to
the scan lands in one place for the whole fleet.

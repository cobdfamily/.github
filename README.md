# cobdfamily/.github

Org-level shared GitHub configuration for the cobdfamily fleet.

## Reusable workflows

Two CVE-scan flavours, picked by whether a published image exists yet:

- [`reusable-cve-scan.yml`](.github/workflows/reusable-cve-scan.yml)
  — **image-based.** Daily Grype scan of a *published* image's
  oras-attached CycloneDX SBOM, SARIF to the Security tab. Callers
  supply only the `image:` input (e.g. `cobdfamily/brl`) and the
  schedule/dispatch trigger. Used by the released **url2code** fleet.

- [`reusable-cve-scan-source.yml`](.github/workflows/reusable-cve-scan-source.yml)
  — **source-based.** Regenerates a CycloneDX SBOM from the source tree
  (python via uv+cyclonedx-py, node via npm+cyclonedx-npm) and Grype-
  scans it. Doesn't need a published image, so it fits **greenfield**
  repos (the **identity** fleet: medici, notaio, atrium, bolla, florin,
  loginwith.email) and **libraries** with no image (medici-webhook-sdk).
  Inputs: `ecosystem` (python|node), `legacy_peer_deps` (node, for
  florin's Angular tree); optional `MEDICI_SDK_TOKEN` secret for python
  repos that pin the private medici-webhook-sdk release wheel. Identity
  repos move to the image-based scan once they cut their first release.

A released-image consumer calls the image-based one like:

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

A greenfield/library consumer calls the source-based one like:

```yaml
# .github/workflows/cve-scan.yml
name: cve-scan
on:
  schedule:
    - cron: "0 11 * * *"
  workflow_dispatch:
permissions:
  contents: read
  security-events: write
  actions: read
jobs:
  cve-scan:
    uses: cobdfamily/.github/.github/workflows/reusable-cve-scan-source.yml@main
    permissions:
      contents: read
      security-events: write
      actions: read
    with:
      ecosystem: python        # or: node
      # legacy_peer_deps: true # node only (florin)
    secrets:
      MEDICI_SDK_TOKEN: ${{ secrets.MEDICI_SDK_TOKEN }}  # atrium, notaio only
```

This replaces the per-repo copy-pasted scan logic so a change to
the scan lands in one place for the whole fleet.

# Cupkek-Games org-level GitHub config

Special [`.github` repository](https://docs.github.com/en/communities/setting-up-your-project-for-healthy-contributions/creating-a-default-community-health-file) for the **Cupkek-Games** GitHub organization. Files placed under `.github/` here are inherited by every repo in the org as fallback defaults.

## Reusable workflows

### [`.github/workflows/upm-release.yml`](.github/workflows/upm-release.yml)

Reusable workflow that packs a CupkekGames Unity package and publishes its tarball + sidecar JSON to the triggering repo's GitHub Release. Consumed by every CupkekGames sibling package's own `.github/workflows/release.yml`:

```yaml
# In each sibling repo (e.g. CupkekGames-ServiceLocator):
#   .github/workflows/release.yml
name: Release
on:
  push:
    tags: ['v*.*.*']

jobs:
  release:
    uses: Cupkek-Games/.github/.github/workflows/upm-release.yml@main
    permissions:
      contents: write
```

To release a sibling package:

1. Bump `package.json > version` in that repo.
2. Commit, tag `v<version>`, push tag.
3. The workflow validates the tag matches `package.json`, packs an npm-style `.tgz`, computes SHA-1 + SHA-512 hashes, and uploads `<name>-<version>.tgz` + `<name>-<version>.json` (sidecar) as Release assets.

The sidecar JSON shape is:

```json
{
  "shasum": "<sha1 hex>",
  "integrity": "sha512-<base64>",
  "packageJson": { "...verbatim package.json..." }
}
```

Read at runtime by the dynamic packument route at `docs.cupkek.games/upm/<package-id>` (lives in [Lix3nn53/luna-docs-next](https://github.com/Lix3nn53/luna-docs-next), file: `src/app/upm/[package]/route.ts`). Together they form the CupkekGames UPM scoped registry.

#### Inputs

- `include-samples` (default `true`) — set to `false` in a sibling's caller workflow if its `Samples~/` is too heavy to ship in the install tarball:
  ```yaml
  jobs:
    release:
      uses: Cupkek-Games/.github/.github/workflows/upm-release.yml@main
      with:
        include-samples: false
  ```

## Pinning

`@main` is convenient while iterating. For stability, pin sibling callers to a tagged version of this repo (e.g. `@v1`) once the workflow stabilizes.

This directory contains workflows that automate the Seqera R-IDE release process, including syncing tags from upstream, applying patches, building binaries, and publishing GitHub releases.

## Workflows

### `seqera-sync-upstream.yml` — Sync Upstream Tags

Syncs tags from the upstream `rstudio/rstudio` repository into this fork (`seqeralabs/r-ide`).

- **Triggers**: Manual (`workflow_dispatch`) or called by another workflow (`workflow_call`).
- **What it does**:
    - Adds `rstudio/rstudio` as the `upstream` remote.
    - Fetches all upstream tags.
    - Pushes tags matching `v20[2-9][0-9]*` (i.e. year-based tags from v2020 onwards) to `origin`.
    - Prints a sorted summary of available tags to the job summary.
- **What it skips**: Tags not matching the year-based pattern (old tags like `v0.x`, `v1.x`). Seqera tags (`seqera-*`) are never present in upstream so are naturally excluded.
- **Output**: Upstream tags available in `seqeralabs/r-ide` for use in the release workflow.

> **Note**: The tag filter relies on the upstream pattern `vYYYY.MM.patch+build` (e.g. `v2025.05.0+496`). If upstream changes its tag format, the filter in the sync step will need updating.

---

### `seqera-validate-patches.yml` — Validate Patches

Validates that all patches in the `PATCH/` folder apply cleanly to a given upstream tag, without creating any branches or releases.

- **Triggers**: Manual (`workflow_dispatch`) or called by another workflow (`workflow_call`).
- **Input**: Upstream tag to validate against (e.g. `v2025.05.0+496`).
- **What it does**: Checks each patch in order with `git apply --check`. If a patch passes, it is applied so subsequent patches have the correct context. Stops at the first failure.
- **Output**: A results table in the job summary showing each patch as ✅ OK or ❌ FAILED. The workflow fails if any patch fails, making it easy to spot in the Actions UI.
- **Use case**: Run this manually before triggering a release to confirm all patches are compatible with a new upstream tag. Also runs automatically as the second step in `seqera-r-ide-release.yml`, blocking the release if patches would fail.

---

### `seqera-r-ide-release.yml` — Build and Release

Automates the full release process for a given upstream tag. The release includes two `.deb` files, one for `amd64` and one for `arm64`.

- **Triggers**: Manual (`workflow_dispatch`) with a required tag input (e.g. `v2025.05.0+496`).
- **Steps**:
    1. **Sync Upstream** — calls `seqera-sync-upstream.yml` to ensure the requested tag is available.
    2. **Validate Patches** — calls `seqera-validate-patches.yml` to confirm all patches apply cleanly. Blocks the release if any patch fails.
    3. **Checkout and Branch Creation** — checks out the tag and creates a branch `seqera/<tag>`.
    4. **Patch Application** — fetches and applies patches from the `PATCH` folder on the default branch.
    5. **Tag and Release Creation** — creates tag `seqera-<tag>` and a GitHub release.
    6. **Binary Build** — builds `.deb` packages for `arm64` and `amd64` and uploads them to the release.
- **Outputs**:
    - New branch: `seqera/<tag>`
    - New tag: `seqera-<tag>`
    - Binaries: `amd64.deb` and `arm64.deb` attached to the GitHub release.

---

### `seqera-r-ide-release-tag.yml` — Rebuild Existing Release

Rebuilds and replaces the binaries for an already-existing Seqera release without creating a new tag or release.

- **Triggers**: Manual (`workflow_dispatch`) with a required Seqera tag input (e.g. `seqera-v2025.05.0+496`).
- **What it does**: Finds the existing release for the given tag, deletes the current assets, and re-uploads freshly built `.deb` files.
- **Use case**: The tag `seqera-<tag>` and GitHub release already exist, but the binary build failed or produced bad artifacts. Use this workflow to rebuild and re-upload the `.deb` files without touching the tag, branch, or release metadata.

> **Note**: This workflow requires the tag `seqera-<tag>` and its GitHub release to already exist. It is **not** a recovery tool for patch application failures — if patching failed, the tag and release were never created, and the full `seqera-r-ide-release.yml` workflow must be re-run instead (after fixing the patches in the `PATCH/` folder and deleting the partially created `seqera/<tag>` branch).

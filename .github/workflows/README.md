This workflow automates the process of creating a new branch, applying patches, building binaries, and creating a 
GitHub release for the Seqera R-IDE project. The release includes two `.deb` files, one for `amd64` and another for `arm64` architecture.

## Overview

The workflow performs the following steps:
1. **Input Argument**: Accepts an existing tag from the forked repository (e.g., `v2025.05.0+496`).
2. **Checkout and Branch Creation**:
    - Checks out the code from the specified tag.
    - Creates a new branch with the format `seqera/<tag>`.
3. **Patch Application**:
    - Fetches patches from the default branch.
    - Applies patches located in the `PATCH` folder.
4. **Tag and Release Creation**:
    - Creates a new tag with the format `seqera-<tag>`.
    - Creates a GitHub release with the new tag.
5. **Binary Build**:
    - Builds binaries for Linux distributions (`noble`) and architectures (`arm64`, `amd64`).
    - Uploads the generated `.deb` files to the GitHub release.

#### 1. Apply Patches, Create Tag, and Release
- **Description**:
    - Checks out the code from the specified tag.
    - Creates a new branch with the format `seqera/<tag>`.
    - Fetches and applies patches from the default branch.
    - Creates a new tag with the format `seqera-<tag>`.
    - Creates a GitHub release with the new tag.
- **Output**: A new branch with the applied patches, a new tag, and a GitHub release.

#### 2. Build Server Linux
- **Description**:
    - Builds binaries for Linux (`noble`) for architectures `arm64` and `amd64`.
    - Uploads the compiled `.deb` files to the GitHub release.
- **Output**: Compiled binaries in `.deb` format uploaded to the release.

## Outputs
- **New Branch**: `seqera/<tag>`.
- **New Tag**: `seqera-<tag>`.
- **Binaries**: Two `.deb` files (`amd64.deb` and `arm64.deb`) uploaded to the GitHub release.
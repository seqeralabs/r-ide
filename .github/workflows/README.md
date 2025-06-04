# Workflow Documentation: Build Server Linux Image Seqera R-IDE

This workflow automates the process of creating a new branch, applying patches, building binaries, and creating a 
GitHub release for the Seqera R-IDE project.

## Overview

The workflow performs the following steps:
1. **Input Argument**: Accepts an existing tag from the forked repository (e.g., `v2025.05.0+496`).
2. **Checkout and Branch Creation**:
    - Checks out the code from the specified tag.
    - Creates a new branch with the format `seqera/<tag>`.
3. **Patch Application**:
    - Fetches patches from the `seqera/test` branch.
    - Applies patches located in the `PATCH` folder.
4. **Tag Creation**: Creates a new tag with the format `seqera-<tag>`.
5. **Binary Build**:
    - Builds binaries for Linux distributions (`noble`) and architectures (`arm64`, `amd64`).
6. **GitHub Release**:
    - Creates a new GitHub release.
    - Uploads the generated binaries to the release.

## Workflow Details

### Input Argument
- **`tag`**: The tag from the forked repository to base the new branch on. Example: `v2025.05.0+496`.

### Jobs

#### 1. Apply Patches and Push Branch
- **Description**:
    - Checks out the code from the specified tag.
    - Creates a new branch with the format `seqera/<tag>`.
    - Fetches and applies patches from the `seqera/test` branch.
- **Output**: A new branch with the applied patches.

#### 2. Build Server Linux
- **Description**:
    - Builds binaries for Linux (`noble`) for architectures `arm64` and `amd64`.
    - Creates a new tag with the format `seqera-<tag>`.
- **Output**: Compiled binaries in `.deb` format.

#### 3. Create GitHub Release
- **Description**:
    - Creates a new GitHub release with the generated tag.
    - Uploads the compiled binaries to the release.

## Binary Build Details
- **Operating System**: Linux (`noble`).
- **Architectures**: `arm64`, `amd64`.

## Outputs
- **New Branch**: `seqera/<tag>`.
- **New Tag**: `seqera-<tag>`.
- **Binaries**: Uploaded to the GitHub release in `.deb` format.

This workflow ensures a streamlined process for building and releasing Seqera R-IDE binaries with applied customizations.
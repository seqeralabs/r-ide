# Change Log

## [seqera-v2025.05.0+496]

### Added
- **`seqera__01_make_package.patch`**: Removed the `dupbuild=warn` flag from the `make-package` script.
    - **Reason**: The flag was causing issues during the server build process.
    - **REMOVED**: Patch not applicable from version `v2026.01.2+418`
- **`seqera__02_logos.patch`**: Replace logos with R language versions
- **`seqera__03_remove_mentions_or_RStudio.patch`**: Remove mentions of RStudio
  - **Warning**: Applicable for `v2025.05.0`, might generate conflicts with other branches
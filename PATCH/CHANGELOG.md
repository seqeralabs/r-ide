# Change Log

## [v2026.01.2+418]

### Restructured
- Split the two monolithic patches (`seqera__02_logos.patch`, `seqera__03_remove_mentions_or_RStudio.patch`) into **84 per-file patches** grouped by prefix:
    - `logos__` — 24 patches (binary asset replacements)
    - `replacer__` — 60 patches (RStudio/Shiny text substitutions in source, properties, and XML files)
- New naming convention: `NNN_<group>__<filename>.patch`. See `README.md` for details.

### Regenerated
- **21 patches** refreshed against v2026 context via 3-way merge using `v2025.05.0+496` as base and `seqera-v2025.05.0+496` as the patched reference.
- **7 patches** manually resolved where upstream content changes overlapped with Seqera's intent:
    - `078_replacer__RsconnectConstants_en.properties.patch`
    - `079_replacer__RsconnectConstants_fr.properties.patch`
    - `080_replacer__ClientWorkbenchConstants_en.properties.patch`
    - `081_replacer__ClientWorkbenchConstants_fr.properties.patch`
    - `082_replacer__PrefsConstants_en.properties.patch`
    - `083_replacer__PrefsConstants_fr.properties.patch`
    - `084_replacer__UserPrefsAccessorConstants_fr.properties.patch`

### Removed
- **`StudioClientProjectConstants.java`** — upstream removed `@DefaultMessage` annotations from this file; the rebranding intent is now covered entirely by the corresponding `.properties` patch.
- **`QuartoConstants.java`** — same reason as above.
- **`ViewsSourceConstants.java`** — same reason as above.
- **`UserStateAccessorConstants_fr.properties`** — after applying the RStudio-stripping transformation to upstream's v2026 content, the resulting file already matched Seqera's intent, producing an empty (no-op) patch.

## [seqera-v2025.05.0+496]

### Added
- **`seqera__01_make_package.patch`**: Removed the `dupbuild=warn` flag from the `make-package` script.
    - **Reason**: The flag was causing issues during the server build process.
    - **REMOVED**: Patch not applicable from version `v2026.01.2+418`
- **`seqera__02_logos.patch`**: Replace logos with R language versions
- **`seqera__03_remove_mentions_or_RStudio.patch`**: Remove mentions of RStudio
  - **Warning**: Applicable for `v2025.05.0`, might generate conflicts with other branches
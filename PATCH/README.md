# PATCH Folder Documentation

This folder contains all meaningful changes made to the forked repository. These patches are designed to be applied 
automatically by the `seqera-r-ide-release` workflow.

## Guidelines for Adding Patches

1. **Purpose**:
    - Patches should only include changes that are necessary to customize the base version of the repository for the Seqera R-IDE.
    - Ensure the changes are meaningful and provide a clear difference from the base version.
    - Default branch (`seqera/main`) should contain both all changes from patch files and patch files themselves.

2. **Workflow Compatibility**:
    - Patches must be structured in a way that allows them to be easily applied by the `seqera-r-ide-release` workflow.
    - Avoid adding changes that could cause conflicts or require manual intervention during the patching process.
    - File naming convention: `NNN_<group>__<filename>.patch`
        - `NNN` — zero-padded global sequence number that determines apply order.
        - `<group>` — logical grouping prefix, currently one of:
            - `logos__` — binary asset replacements (icons, logos, favicons).
            - `replacer__` — text substitutions to remove or rebrand RStudio/Shiny mentions in source, properties, and XML files.
        - `<filename>` — the basename of the single file the patch modifies.
    - Patches are split **one per modified file**. This means when a patch fails during validation, the failure pinpoints the exact file needing attention, without blocking the other unrelated patches from being analyzed.

3. **Content**:
    - Include only modifications that are essential for the functionality or customization of the Seqera R-IDE.
    - Do not include unrelated or redundant changes.
    - Each patch should touch only one file. If a logical change spans multiple files, create one patch per file.

4. **Testing**:
    - Use the `seqera-validate-patches.yml` GitHub Actions workflow to validate all patches against any upstream tag before releasing. It runs `git apply --check` on each patch in order and reports a results table showing which apply cleanly and which fail.
    - The same validation runs automatically as a blocking step in `seqera-r-ide-release.yml`.

## Notes
- Patches do not include created GitHub workflows. For information about workflows and their documentation, refer to the `.github/workflows` directory.
- Maintain a clear and concise commit message for each patch to document its purpose and changes.

By following these guidelines, future developers can ensure that patches remain manageable, relevant, and compatible with the automated workflow.
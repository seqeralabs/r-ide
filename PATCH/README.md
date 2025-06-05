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
    - To ensure the order of patches, please follow the file naming convention `seqera__<number>_<description>.patch`

3. **Content**:
    - Include only modifications that are essential for the functionality or customization of the Seqera R-IDE.
    - Do not include unrelated or redundant changes.

4. **Testing**:
    - Test patches thoroughly to ensure they apply cleanly to the base version and do not introduce errors.

## Notes
- Patches do not include created GitHub workflows. For information about workflows and their documentation, refer to the `.github/workflows` directory.
- Maintain a clear and concise commit message for each patch to document its purpose and changes.

By following these guidelines, future developers can ensure that patches remain manageable, relevant, and compatible with the automated workflow.
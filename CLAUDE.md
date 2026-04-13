# CLAUDE.md

Guidance for working with this repository.

## Repo overview

This is Seqera's fork of an external upstream repo, used to build a customized Seqera R-IDE distribution.

- **Upstream:** the external project this repo is forked from (configured as the `parent` on GitHub)
- **Fork (origin):** `seqeralabs/r-ide`
- **Source of truth branch:** `seqera/main` — contains the `PATCH/` folder and `.github/workflows/`. This branch does *
  *not** have the customizations applied to source files; it only holds the patches.
- **Per-release branches:** `seqera/<upstream-tag>` — created by the release workflow, contains upstream source with
  patches applied.

## Branch and tag conventions

| Name pattern                                       | Meaning                                                          | Safe to modify?             |
|----------------------------------------------------|------------------------------------------------------------------|-----------------------------|
| `seqera/main`                                      | Source of truth for patches and workflows                        | Yes (via PR review)         |
| `seqera/<tag>` (e.g. `seqera/v2026.01.2+418`)      | Auto-created by the release workflow; upstream + patches applied | No — auto-generated         |
| `seqera-<tag>` (tag, e.g. `seqera-v2026.01.2+418`) | Git tag for a Seqera release                                     | No — auto-generated         |
| `v20XX.*` (tag, e.g. `v2026.01.2+418`)             | Upstream tag mirrored into the fork by the sync workflow         | No — mirrored from upstream |
| Any other upstream branch                          | Mirrored from upstream repo                                      | No                          |

**Do not touch** anything matching `seqera/` (branches) or `seqera-` (tags) other than `seqera/main`. The release
automation depends on the exact naming.

## `PATCH/` folder conventions

Patches in `PATCH/` are applied in alphabetical order by the release workflow.

**Naming:** `NNN_<group>__<filename>.patch`

- `NNN` — zero-padded 3-digit global sequence number determining apply order
- `<group>` — logical grouping:
    - `logos__` — binary asset replacements (icons, PNGs, SVGs, favicons)
    - `replacer__` — text substitutions (strip upstream brand names, rebrand to Seqera intent) in source, properties,
      and XML files
- `<filename>` — basename of the single file the patch modifies

**Rules:**

- **One patch per file.** If a logical change spans multiple files, create one patch per file. This makes failures
  pinpoint the exact file needing attention.
- Keep patches minimal — only include lines that differ from upstream.
- Document meaningful additions in `PATCH/CHANGELOG.md` under a version section.

See `PATCH/README.md` for the full policy and `PATCH/CHANGELOG.md` for history.

## Workflow map

Defined in `.github/workflows/`. See `.github/workflows/README.md` for details.

- **`seqera-sync-upstream.yml`** — syncs year-based upstream tags (`v20XX.*`) from the upstream repo into origin. Manual
  or called from the release workflow.
- **`seqera-validate-patches.yml`** — runs `git apply --check --ignore-whitespace` on every patch in order against a
  given upstream tag. Reports a pass/fail table. Manual or called from the release workflow. **Does not modify the repo
  ** — validation only.
- **`seqera-r-ide-release.yml`** — full release: `sync-upstream` → `validate-patches` → `apply-patches` (creates
  `seqera/<tag>` branch and `seqera-<tag>` tag) → `build-server-linux` (`.deb` packages for amd64 and arm64).
- **`seqera-r-ide-release-tag.yml`** — rebuilds binaries for an existing release; use only when the release and tag
  already exist and the binary build failed.

Both `apply` steps use `git apply --ignore-whitespace` so that whitespace and line-ending mismatches (e.g. an LF patch
targeting a CRLF file) do not cause false failures. See the **Line endings** note below.

## Patch conflict resolution playbook

When the validate-patches workflow (or manual `git apply --check`) reports failures against a new upstream tag, follow
this process.

### 1. Set up a working branch

```bash
git fetch origin --tags
git checkout -b tmp/<new-tag> <new-tag>   # e.g. v2026.01.2+418
```

Patches in `PATCH/` live on `seqera/main`, so fetch them from there as needed with
`git show seqera/main:PATCH/<name>.patch` or `git checkout seqera/main -- PATCH`.

### 2. Identify the previous release's base

Each release's patches were originally written against a specific upstream tag. Find it by locating the `Apply patch`
commit on the previous `seqera/<old-tag>` branch:

```bash
git log --all --oneline --grep='Apply patch' -5
```

The parent of that commit is the base upstream tag (e.g. `v2025.05.0+496`). The tag `seqera-<old-tag>` contains the
corresponding patched state.

### 3. Test each patch

```bash
for p in PATCH/*.patch; do
  git apply --check --ignore-whitespace "$p" 2>/dev/null && echo "OK   $p" || echo "FAIL $p"
done
```

### 4. For failing patches — attempt 3-way merge

For each failing patch's target file:

```bash
FILE="path/to/file"
OLD_TAG="v2025.05.0+496"            # previous base upstream tag
SEQERA_OLD="seqera-v2025.05.0+496"  # previous patched state

git show "$OLD_TAG:$FILE" > /tmp/base.txt
git show "$SEQERA_OLD:$FILE" > /tmp/ours.txt
cp /tmp/ours.txt /tmp/merged.txt

# 3-way merge: apply upstream changes (base → current) on top of patched state
git merge-file -p /tmp/merged.txt /tmp/base.txt "$FILE" > /tmp/result.txt
```

- **Exit code 0** → clean merge. Replace the file with `/tmp/result.txt`, then regenerate the patch via
  `git diff -- "$FILE"`.
- **Exit code > 0** → conflicts. See step 5.

### 5. Resolve remaining conflicts

Conflict blocks appear as `<<<<<<< ours === theirs >>>>>>>`. The standard rule:

> **Take "theirs" (the new upstream content) and apply the Seqera transformation to it.**

The transformations applied by `replacer__` patches are textual substitutions (see the next section). If upstream
removed content that Seqera had added or vice versa, accept upstream's decision unless the removal breaks Seqera
functionality.

### 6. Drop obsolete patches

A patch should be dropped (not forced) when:

- **Upstream removed the target content entirely** — e.g. refactored out auto-generated default messages; the intent is
  now covered elsewhere (usually a `.properties` patch).
- **Post-resolution the patch is empty** — upstream's new content already matches Seqera intent after applying
  transformations; there's nothing left to diff.
- **The original rationale no longer applies** — e.g. upstream fixed the bug the patch was working around.

Record every drop in `PATCH/CHANGELOG.md` under the relevant version, with a one-line reason.

### 7. Line endings (CRLF/LF)

Some files in the upstream repo have **CRLF line endings** (Windows style) while patches from various sources may have
LF-only content. `git apply` on Linux CI is strict about this mismatch; macOS is lenient.

**This is why the workflows run `git apply --ignore-whitespace`** — it makes line-ending differences non-blocking.

If regenerating a patch manually:

- Prefer `git diff` against the actual checked-out file — it preserves the file's line endings in the patch.
- **Do not** run the patch through whitespace-normalizing tools (editors that auto-convert on save, `dos2unix`, etc.) —
  that can silently rewrite entire files when Git re-diffs them.
- To confirm CRLF status of a file: `file <path>` will print `with CRLF line terminators` if present.

### 8. Verify end-to-end

After regenerating patches, apply them all in sequence on a clean checkout of the new upstream tag:

```bash
git checkout <new-tag> -- .
for p in PATCH/*.patch; do git apply --ignore-whitespace "$p" || echo "FAIL: $p"; done
```

Whitespace warnings from `git apply` are expected for `replacer__` patches (stripped words leave trailing spaces) — not
errors.

## Common transformations (replacer__ patches)

Standard substitutions used to remove upstream brand/product names. Order matters — longest phrase first.

**English:** strip the brand name after prepositions (`on ...`, `by ...`, `from ...`, `to ...`) and standalone;
product-specific suffixes (`Server`, `Desktop`, `Pro`); rebrand the generic IDE name to `IDE`; replace `ShinyApps.io`
with `Apps.io`; strip `Shiny` from "Shiny applications" phrasing.

**French:** strip the brand name after the equivalent prepositions (`par ...`, `de ...`, `sur ...`) and standalone.

After substitution, collapse double spaces (`  ` → ` `) and fix punctuation spacing (` .` → `.`).

Inspect the existing `replacer__*.patch` files for the exact string substitutions currently in use before adding new
ones.

## Release flow (high level)

1. **Sync**: run `seqera-sync-upstream.yml` to fetch the new upstream tag into origin.
2. **Validate**: run `seqera-validate-patches.yml` with the new tag. If any patch fails, fix it on `seqera/main` (see
   the playbook above) before proceeding.
3. **Release**: run `seqera-r-ide-release.yml` with the new tag. This re-runs sync + validate as gates, then applies
   patches, creates `seqera/<tag>` + `seqera-<tag>`, and builds the `.deb` packages.
4. **Rebuild only** (rare): if the release exists but the binary build failed, run `seqera-r-ide-release-tag.yml` with
   the Seqera tag name.

## Working style

- **Present a plan before editing files.** For non-trivial changes, describe the approach and wait for approval before
  modifying anything in the repo. Use `/tmp/` scratch areas for exploration.
- **Ask when confused.** Prefer a clarifying question over guessing.
- **Keep changes scoped.** Don't refactor unrelated code alongside a fix.
- **Do not regenerate patches by running them through `git apply` + `git diff` blindly** — if local Git has different
  whitespace/line-ending handling than the file, `git diff` can produce a patch that rewrites the entire file. Prefer
  small targeted edits or 3-way merge (see playbook step 4).

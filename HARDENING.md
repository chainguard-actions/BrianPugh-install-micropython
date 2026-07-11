<!-- markdownlint-disable -->

# Hardening Report: BrianPugh--install-micropython/v3.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **BrianPugh--install-micropython/v3.0.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): ${{ }} expressions are directly interpolated inside run: shell commands. In build-and-pack.yml, `${{ env.PACKED_JS_PATH }}` (a workflow-controlled env var) and `${{ github.ref_name }}` (GitHub context) are interpolated unquoted in shell commands such as `git status ${{ env.PACKED_JS_PATH }}`, `git add ${{ env.PACKED_JS_PATH }}`, and `git push origin HEAD:${{ github.ref_name }}`. Any of these values could contain shell metacharacters.

Locations:

- `.github/workflows/build-and-pack.yml:31`

### script-injection (severity: high)

Rule (a): ${{ env.TAG_NAME }} is directly interpolated inside a run: shell command block. TAG_NAME is set from `github.event.inputs.TAG_NAME` (a user-controlled workflow_dispatch input) or `github.event.release.tag_name`. It is interpolated in commands such as `echo "${{ env.TAG_NAME }}"`, `git rev-parse "${{ env.TAG_NAME }}"`, and `echo "Updating $MAJOR_TAG to point to ${{ env.TAG_NAME }}"`. An attacker with write access could supply a malicious tag name containing shell metacharacters.

Locations:

- `.github/workflows/release-new-action-version.yml:50`

### script-injection (severity: high)

Rule (a): ${{ }} expressions are directly interpolated inside run: shell commands in test.yaml. `${{ steps.install.outputs.sha }}` is interpolated in echo and conditional commands (lines ~50-55). `${{ steps.install.outputs.cache-hit }}` and `${{ matrix.submodules }}` are interpolated in conditional shell expressions (lines ~87-100). Step outputs and matrix values flow through YAML template substitution before the shell sees them, making them potential injection vectors.

Locations:

- `.github/workflows/test.yaml:50`
- `.github/workflows/test.yaml:87`
- `.github/workflows/test.yaml:97`

### unpinned-uses (severity: high)

All uses: references across the workflow files use mutable tag-based refs (@v5) instead of immutable full 40-character SHA commit hashes. This exposes the workflows to supply-chain attacks if the referenced action tags are moved or compromised. Failing references: actions/checkout@v5, actions/setup-node@v5 (in build-and-pack.yml, release-new-action-version.yml, and test.yaml).

Locations:

- `.github/workflows/build-and-pack.yml:17`
- `.github/workflows/build-and-pack.yml:20`
- `.github/workflows/release-new-action-version.yml:23`
- `.github/workflows/release-new-action-version.yml:27`
- `.github/workflows/release-new-action-version.yml:44`
- `.github/workflows/test.yaml:25`
- `.github/workflows/test.yaml:28`
- `.github/workflows/test.yaml:73`
- `.github/workflows/test.yaml:110`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all findings across three workflow files:

1. build-and-pack.yml:
   - Pinned actions/checkout@v5 to SHA 93cb6efe18208431cddfb8368fd83d5badbf9bfd
   - Pinned actions/setup-node@v5 to SHA a0853c24544627f65ddf259abe73b1d18a591444
   - Moved ${{ env.PACKED_JS_PATH }} and ${{ github.ref_name }} into step env: block; referenced as $PACKED_JS_PATH and $REF_NAME in shell

2. release-new-action-version.yml:
   - Pinned actions/checkout@v5 to SHA 93cb6efe18208431cddfb8368fd83d5badbf9bfd (two occurrences)
   - Pinned actions/setup-node@v5 to SHA a0853c24544627f65ddf259abe73b1d18a591444
   - Moved ${{ env.TAG_NAME }} into step env: block; referenced as $TAG_NAME in shell

3. test.yaml:
   - Pinned actions/checkout@v5 to SHA 93cb6efe18208431cddfb8368fd83d5badbf9bfd (three occurrences)
   - Pinned actions/setup-node@v5 to SHA a0853c24544627f65ddf259abe73b1d18a591444
   - Moved ${{ steps.install.outputs.sha }} into env: block as INSTALL_SHA
   - Moved ${{ steps.install.outputs.cache-hit }} into env: block as CACHE_HIT
   - Moved ${{ matrix.submodules }} into env: block as SUBMODULES


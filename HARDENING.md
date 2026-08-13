<!-- markdownlint-disable -->

# Hardening Report: BrianPugh--install-micropython/v2.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **BrianPugh--install-micropython/v2.2.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Workflow files reference actions using mutable tags or branch names instead of pinned full-length commit SHAs, making them vulnerable to supply-chain attacks.

build-and-pack.yml:
  - uses: actions/checkout@v4
  - uses: actions/setup-node@v4
  - uses: ad-m/github-push-action@master  (branch ref — especially dangerous)

test.yaml:
  - uses: actions/checkout@v4
  - uses: actions/setup-node@v4

release-new-action-version.yml:
  - uses: actions/publish-action@v0.2.2

All should be pinned to a full 40-character hex commit SHA (e.g. actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4).

Locations:

- `.github/workflows/build-and-pack.yml:14`
- `.github/workflows/build-and-pack.yml:17`
- `.github/workflows/build-and-pack.yml:40`
- `.github/workflows/test.yaml:18`
- `.github/workflows/test.yaml:22`
- `.github/workflows/release-new-action-version.yml:18`

### script-injection (severity: high)

Two run: steps in build-and-pack.yml directly interpolate ${{ env.PACKED_JS_PATH }} inside shell command strings. Any ${{ ... }} expression interpolated into a run: block is a script-injection risk (sub-rule a) because the value is substituted by the template engine before the shell parses it, allowing shell metacharacters to be injected.

Offending lines:
  1. echo "changes=$(git status ${{ env.PACKED_JS_PATH }} --porcelain)" >> $GITHUB_OUTPUT
  2. git add ${{ env.PACKED_JS_PATH }}
  3. git commit -m "Pack with dependencies to ${{ env.PACKED_JS_PATH }}"

Fix: move PACKED_JS_PATH into an env: block and reference it as a quoted shell variable ("$PACKED_JS_PATH") inside the run: script.

Locations:

- `.github/workflows/build-and-pack.yml:27`
- `.github/workflows/build-and-pack.yml:33`

### github-env-injection (severity: high)

The 'Check packed js changes' step in build-and-pack.yml writes a value derived from the ${{ env.PACKED_JS_PATH }} expression directly into $GITHUB_OUTPUT without the required sanitization step (printf '%s' ... | tr -d '\n\r'). An attacker who can influence PACKED_JS_PATH could inject newlines to poison GITHUB_OUTPUT.

Offending line:
  echo "changes=$(git status ${{ env.PACKED_JS_PATH }} --porcelain)" >> $GITHUB_OUTPUT

Fix: sanitize the value before writing, e.g.:
  safe=$(printf '%s' "$PACKED_JS_PATH" | tr -d '\n\r')
  echo "changes=$(git status "$safe" --porcelain)" >> $GITHUB_OUTPUT

Locations:

- `.github/workflows/build-and-pack.yml:27`

### missing-permissions (severity: medium)

Two workflow files have no top-level permissions: key and no job-level permissions: key on any of their jobs. Without explicit permissions, workflows inherit the default repository permissions (which may be read/write), violating the principle of least privilege.

- build-and-pack.yml: no permissions declared at top level or job level
- test.yaml: no permissions declared at top level or job level

Add a top-level permissions: block with only the minimum required scopes (e.g. contents: read for read-only workflows, or contents: write only where needed).

Locations:

- `.github/workflows/build-and-pack.yml:1`
- `.github/workflows/test.yaml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, missing-permissions

**Notes:**

Fixed all 4 findings across 3 workflow files:

1. **unpinned-uses**: Pinned all 6 action references to full commit SHAs: actions/checkout@v4→11d5960a..., actions/setup-node@v4→49933ea5..., ad-m/github-push-action@master→881a6320..., actions/publish-action@v0.2.2→dca2315f...

2. **script-injection**: Moved ${{ env.PACKED_JS_PATH }} out of run: shell strings into step-level env: blocks in both the 'Check packed js changes' and 'Commit packed js' steps, referencing as $PACKED_JS_PATH shell variable.

3. **github-env-injection**: Added sanitization in 'Check packed js changes' step using `safe=$(printf '%s' "$PACKED_JS_PATH" | tr -d '\n\r')` before using the value in git status and writing to GITHUB_OUTPUT.

4. **missing-permissions**: Added `permissions: contents: write` to build-and-pack.yml (needs write to commit/push packed JS) and `permissions: contents: read` to test.yaml (read-only access sufficient for testing).


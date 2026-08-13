<!-- markdownlint-disable -->

# Hardening Report: BrianPugh--install-micropython/v2.4.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **BrianPugh--install-micropython/v2.4.0** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation inside run: blocks. In build-and-pack.yml, `${{ env.PACKED_JS_PATH }}` is interpolated directly into shell commands: `echo "changes=$(git status ${{ env.PACKED_JS_PATH }} --porcelain)" >> $GITHUB_OUTPUT`, `git add ${{ env.PACKED_JS_PATH }}`, and `git commit -m "Pack with dependencies to ${{ env.PACKED_JS_PATH }}"`. Although `env.PACKED_JS_PATH` is set in the job env block, any `${{ ... }}` expression inside a run: block is a script-injection risk because YAML template substitution happens before the shell ever sees the value. The value should be referenced as the shell variable `$PACKED_JS_PATH` instead.

Locations:

- `.github/workflows/build-and-pack.yml:30`
- `.github/workflows/build-and-pack.yml:36`
- `.github/workflows/build-and-pack.yml:37`

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation inside run: blocks. In release-new-action-version.yml, `${{ env.TAG_NAME }}` (which is sourced from the user-controlled `github.event.inputs.TAG_NAME` or `github.event.release.tag_name`) is interpolated directly into multiple shell commands: `MAJOR_TAG=$(echo "${{ env.TAG_NAME }}" | grep -oE ...)`, `echo "Updating $MAJOR_TAG to point to ${{ env.TAG_NAME }}"`, `COMMIT_SHA=$(git rev-parse "${{ env.TAG_NAME }}")`, and `echo "Source tag ${{ env.TAG_NAME }} points to $COMMIT_SHA"`. An attacker-controlled tag name containing shell metacharacters could achieve command injection. The env var `$TAG_NAME` should be referenced as a shell variable instead.

Locations:

- `.github/workflows/release-new-action-version.yml:31`
- `.github/workflows/release-new-action-version.yml:32`
- `.github/workflows/release-new-action-version.yml:35`
- `.github/workflows/release-new-action-version.yml:36`

### github-env-injection (severity: high)

In build-and-pack.yml, the run: block writes to $GITHUB_OUTPUT using a value derived from the expression `${{ env.PACKED_JS_PATH }}` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). The offending line is: `echo "changes=$(git status ${{ env.PACKED_JS_PATH }} --porcelain)" >> $GITHUB_OUTPUT`. The expression is substituted by the YAML template engine before the shell runs, so a newline in the value could inject additional key=value pairs into GITHUB_OUTPUT.

Locations:

- `.github/workflows/build-and-pack.yml:30`

### unpinned-uses (severity: high)

Multiple workflow files reference actions by mutable tags or branch names instead of full 40-character commit SHAs, making them vulnerable to supply-chain attacks if the referenced tag or branch is moved or compromised. Unpinned references found:
- build-and-pack.yml: `actions/checkout@v4` (line 15), `actions/setup-node@v4` (line 17), `ad-m/github-push-action@master` (line 41) — the `@master` branch reference is especially dangerous.
- release-new-action-version.yml: `actions/checkout@v4` (line 24).
- test.yaml: `actions/checkout@v4` (line 17), `actions/setup-node@v4` (line 20).
All should be pinned to full SHA digests, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `.github/workflows/build-and-pack.yml:15`
- `.github/workflows/build-and-pack.yml:17`
- `.github/workflows/build-and-pack.yml:41`
- `.github/workflows/release-new-action-version.yml:24`
- `.github/workflows/test.yaml:17`
- `.github/workflows/test.yaml:20`

### missing-permissions (severity: medium)

build-and-pack.yml has no top-level `permissions:` key and its single job (`build`) also has no job-level `permissions:` key. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad (write access to contents, packages, etc.). A minimal permissions block such as `permissions: contents: write` should be added.

Locations:

- `.github/workflows/build-and-pack.yml:1`

### missing-permissions (severity: medium)

test.yaml has no top-level `permissions:` key and its single job (`test`) also has no job-level `permissions:` key. Without explicit permissions, the workflow inherits the repository's default token permissions. A minimal permissions block such as `permissions: contents: read` should be added.

Locations:

- `.github/workflows/test.yaml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all 6 findings across 3 workflow files:

1. build-and-pack.yml:
   - Added top-level `permissions: contents: write`
   - Pinned `actions/checkout@v4` to SHA `11d5960a326750d5838078e36cf38b85af677262`
   - Pinned `actions/setup-node@v4` to SHA `49933ea5288caeca8642d1e84afbd3f7d6820020`
   - Pinned `ad-m/github-push-action@master` to SHA `881a6320fdb16eb5318c5054f31c218aec2b324c`
   - Replaced all `${{ env.PACKED_JS_PATH }}` in run: blocks with shell variable `$PACKED_JS_PATH`
   - Fixed github-env-injection: sanitized path with `printf '%s' "$PACKED_JS_PATH" | tr -d '\n\r'` before writing to GITHUB_OUTPUT

2. release-new-action-version.yml:
   - Pinned `actions/checkout@v4` to SHA `11d5960a326750d5838078e36cf38b85af677262`
   - Replaced all `${{ env.TAG_NAME }}` in run: block with shell variable `$TAG_NAME`

3. test.yaml:
   - Added top-level `permissions: contents: read`
   - Pinned `actions/checkout@v4` to SHA `11d5960a326750d5838078e36cf38b85af677262`
   - Pinned `actions/setup-node@v4` to SHA `49933ea5288caeca8642d1e84afbd3f7d6820020`


<!-- markdownlint-disable -->

# Hardening Report: isort--isort-action/v1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **isort--isort-action/v1** was hardened automatically. 7 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Two `run:` blocks in action.yml directly interpolate `${{ inputs.* }}` expressions into shell command strings (sub-rule a), enabling script injection by any caller of this composite action.

1. The install_packages step (around line 76) interpolates `${{ inputs.isort-version || inputs.isortVersion }}` and `${{ inputs.requirements-files || inputs.requirementsFiles }}` directly into the shell command. An attacker-controlled input value containing shell metacharacters (`;`, `|`, `$(...)`, etc.) would be executed by the shell.

2. The run-isort step (around line 83) interpolates `${{ inputs.configuration }}` and `${{ inputs.sort-paths || inputs.sortPaths }}` directly into the shell command. The same injection risk applies.

Fix: Move all input values into `env:` variables and reference them as double-quoted shell variables (e.g., `"$ISORT_VERSION"`) inside the `run:` block.

Locations:

- `action.yml:76`
- `action.yml:83`

### unpinned-uses (severity: high)

Both workflow files reference `actions/checkout@v4` using a mutable version tag instead of a pinned 40-character commit SHA. A tag can be moved to point to a different (potentially malicious) commit at any time, creating a supply-chain risk.

Failing references:
- `.github/workflows/lint.yaml`: `uses: actions/checkout@v4`
- `.github/workflows/test.yaml`: `uses: actions/checkout@v4`

Fix: Pin each reference to a full SHA, e.g. `uses: actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `.github/workflows/lint.yaml:9`
- `.github/workflows/test.yaml:13`

### missing-permissions (severity: medium)

Neither `.github/workflows/lint.yaml` nor `.github/workflows/test.yaml` declares a `permissions:` block at the top level or at the job level. Without explicit permissions, GitHub Actions grants the default token permissions (which may include `write` access to repository contents depending on organisation settings), violating the principle of least privilege.

Fix: Add a top-level `permissions: {}` (or specific minimal scopes such as `contents: read`) to each workflow file.

Locations:

- `.github/workflows/lint.yaml:1`
- `.github/workflows/test.yaml:1`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.isort-version || inputs.isortVersion }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:79`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.requirements-files || inputs.requirementsFiles }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:79`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.configuration }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:87`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.sort-paths || inputs.sortPaths }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:87`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all 7 findings across 3 files:

1. action.yml (script-injection / static-inline-injection): Moved all ${{ inputs.* }} expressions from run: blocks into env: blocks. The install_packages step now uses ISORT_VERSION and REQUIREMENTS_FILES env vars passed as quoted arguments. The run_isort step uses ISORT_CONFIGURATION (split into array with read -ra) and ISORT_SORT_PATHS env vars, preventing shell injection.

2. .github/workflows/lint.yaml and .github/workflows/test.yaml (unpinned-uses): Pinned actions/checkout@v4 to full commit SHA 11d5960a326750d5838078e36cf38b85af677262 with # v4 comment.

3. .github/workflows/lint.yaml and .github/workflows/test.yaml (missing-permissions): Added top-level `permissions: {}` to both workflow files to enforce least privilege.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two script injection vulnerabilities in .github/workflows/test.yaml:
1. Line 23 (assert-sorted-imports-return-no-output step): Moved `${{ steps.imports-properly-sorted.outputs.isort-result }}` into an `env:` block as `ISORT_RESULT`, and replaced the inline interpolation with `"$ISORT_RESULT"` in the shell script.
2. Line 31 (assert-improperly-sorted-imports-return-output step): Moved `${{ steps.imports-improperly-sorted.outputs.isort-result }}` into an `env:` block as `ISORT_RESULT`, and replaced the inline interpolation with `"$ISORT_RESULT"` in the shell script.
Both values are now passed through the environment rather than being interpolated directly into the shell command string, preventing shell metacharacter injection.


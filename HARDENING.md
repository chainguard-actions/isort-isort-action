<!-- markdownlint-disable -->

# Hardening Report: isort--isort-action/v1.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **isort--isort-action/v1.0.0** was hardened automatically. 7 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Three run: steps in action.yml directly interpolate ${{ inputs.* }} expressions into shell command strings (sub-rule a). The values inputs.isortVersion, inputs.requirementsFiles, inputs.configuration, and inputs.sortPaths are all attacker-controllable (required: false, caller-supplied) and are substituted by the YAML template engine before the shell parses the command, enabling arbitrary command injection. For example: `${{ github.action_path }}/bin/install_packages ${{ inputs.isortVersion }} ${{ inputs.requirementsFiles }}` and `${{ github.action_path }}/bin/run_isort ${{ inputs.configuration }} ${{ inputs.sortPaths }}`. These inputs must be moved into env: variables and the shell expansions must be double-quoted.

Locations:

- `action.yml:31`
- `action.yml:33`
- `action.yml:37`

### unpinned-uses (severity: high)

The workflow uses actions/checkout@v3, which is a mutable tag reference rather than a pinned 40-character commit SHA. A tag can be moved to point to a different (potentially malicious) commit. It should be pinned to a full SHA, e.g. actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v3.

Locations:

- `.github/workflows/lint.yaml:9`

### missing-permissions (severity: medium)

The workflow file lint.yaml has no top-level permissions: key and the single job (lint) also has no job-level permissions: key. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad (e.g. write access to contents). A minimal permissions block such as `permissions: read-all` or specific scopes should be added.

Locations:

- `.github/workflows/lint.yaml:1`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.isortVersion }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:40`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.requirementsFiles }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:40`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.configuration }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:45`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.sortPaths }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:45`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, unpinned-uses, missing-permissions

**Notes:**

1. action.yml: Moved all ${{ inputs.* }} expressions to env: blocks. List inputs (requirementsFiles, configuration, sortPaths) are tokenized with xargs into bash arrays for safe multi-argument handling. Single-value input (isortVersion) is passed as a double-quoted env var. ${{ github.action_path }} is retained inline as it is a trusted GitHub context value. 2. bin/install_packages: Updated to accept isort_version as $1 and remaining args as individual requirement files (using shift + $@), matching the new calling convention. 3. .github/workflows/lint.yaml: Pinned actions/checkout@v3 to full SHA a37ce9120846195fa4ece8f58b268e6043cb2f26 # v3. Added top-level permissions: contents: read block.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Replaced all three occurrences of `${{ github.action_path }}` in `run:` blocks with the pre-set environment variable `$GITHUB_ACTION_PATH`:
- Line 34: `run: ${{ github.action_path }}/bin/ensure_python` → `run: "$GITHUB_ACTION_PATH/bin/ensure_python"`
- Line 44: `"${{ github.action_path }}/bin/install_packages"` → `"$GITHUB_ACTION_PATH/bin/install_packages"`
- Line 57: `"${{ github.action_path }}/bin/run_isort"` → `"$GITHUB_ACTION_PATH/bin/run_isort"`

Using `$GITHUB_ACTION_PATH` avoids YAML template substitution entirely, so the value never flows through the `${{ }}` expression engine before reaching the shell.


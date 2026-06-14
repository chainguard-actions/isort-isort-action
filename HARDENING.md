<!-- markdownlint-disable -->

# Hardening Report: isort--isort-action/v1.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **isort--isort-action/v1.0.0** was hardened automatically. 5 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `run:` steps in action.yml directly interpolate `${{ inputs.* }}` expressions into shell command strings, violating rule (a). This allows an attacker who controls the calling workflow's inputs to inject arbitrary shell commands.

- Step 2 (line 36): `${{ github.action_path }}/bin/install_packages ${{ inputs.isortVersion }} ${{ inputs.requirementsFiles }}` — both `inputs.isortVersion` and `inputs.requirementsFiles` are interpolated directly into the shell command.
- Step 3 (line 41): `${{ github.action_path }}/bin/run_isort ${{ inputs.configuration }} ${{ inputs.sortPaths }}` — both `inputs.configuration` and `inputs.sortPaths` are interpolated directly into the shell command.

Fix: route each input through an `env:` block and reference the env var (double-quoted) inside the `run:` script instead of using `${{ inputs.* }}` directly in the shell string.

Locations:

- `action.yml:36`
- `action.yml:41`

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

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Fixed all script injection findings in action.yml by moving ${{ inputs.* }} expressions into env: blocks for both affected steps:

1. Step 2 (install_packages): Added env: block with ISORT_VERSION=${{ inputs.isortVersion }} and REQUIREMENTS_FILES=${{ inputs.requirementsFiles }}. Shell script now uses "$ISORT_VERSION" and ${REQUIREMENTS_FILES:+"$REQUIREMENTS_FILES"} (conditional expansion for the optional empty-default input).

2. Step 3 (run-isort): Added env: block with CONFIGURATION=${{ inputs.configuration }} and SORT_PATHS=${{ inputs.sortPaths }}. Shell script now uses $CONFIGURATION (unquoted for word splitting of multi-flag values like '--check-only --diff') and "$SORT_PATHS" (quoted).

The ${{ github.action_path }} expression remains in the run: block as it is a GitHub context value used to construct the script path, not a user-controlled input.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed both script injection findings in action.yml:
1. (Finding a) Moved `${{ github.action_path }}` out of all three `run:` shell strings and into `env:` blocks as `ACTION_PATH: ${{ github.action_path }}`. Each step now references `$ACTION_PATH` as a plain environment variable.
2. (Finding b) Fixed unquoted `$CONFIGURATION` expansion in the run-isort step. Instead of passing it unquoted (allowing shell metacharacter injection), the fix uses `read -ra CONF_ARGS <<< "$CONFIGURATION"` to safely word-split the configuration flags into an array, then passes `"${CONF_ARGS[@]}"` to the run_isort script. This preserves multi-flag support (e.g., `--check-only --diff`) while preventing injection attacks.


<!-- markdownlint-disable -->

# Hardening Report: isort--isort-action/v0.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **isort--isort-action/v0.1.0** was hardened automatically. 6 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Three `run:` blocks in action.yml directly interpolate `${{ ... }}` expressions into shell command strings. Line 35 and 37 interpolate attacker-controlled `inputs.*` values (`inputs.isortVersion`, `inputs.requirementsFiles`, `inputs.configuration`, `inputs.sortPaths`) directly into the shell command, enabling command injection. Line 33 interpolates `github.action_path`. All three violate the rule that no `${{ ... }}` expression should appear inside a `run:` shell command string. Fix: move values into `env:` variables and reference those variables (quoted) in the script instead.

Offending lines:
- Line 33: `run: ${{ github.action_path }}/bin/ensure_python`
- Line 35: `run: ${{ github.action_path }}/bin/install_packages "${{ inputs.isortVersion }}" "${{ inputs.requirementsFiles }}"`
- Line 37: `run: ${{ github.action_path }}/bin/run_isort "${{ inputs.configuration }}" "${{ inputs.sortPaths }}"`

Locations:

- `action.yml:33`
- `action.yml:35`
- `action.yml:37`

### script-injection (severity: high)

Sub-rule (b): In `bin/install_packages`, the shell variables `$isort_version` and `$requirements_files` (both sourced from `inputs.isortVersion` and `inputs.requirementsFiles` respectively, which are attacker-controlled) are used unquoted. Specifically: `pip3 install isort[...]==$isort_version` (unquoted), `for file in $requirements_files` (unquoted — allows word-splitting and glob expansion), and `pip install -r $file` (unquoted). An attacker can inject shell metacharacters via these inputs to execute arbitrary commands.

In `bin/run_isort`, `$configuration` and `$sort_paths` (sourced from `inputs.configuration` and `inputs.sortPaths`) are used unquoted in `isort $configuration $sort_paths`, allowing shell metacharacter injection.

Locations:

- `bin/install_packages:8`
- `bin/install_packages:12`
- `bin/install_packages:15`
- `bin/run_isort:5`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.isortVersion }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:37`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.requirementsFiles }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:37`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.configuration }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:40`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.sortPaths }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:40`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Fixed all script injection findings across three files:

1. action.yml: Moved all ${{ }} expressions out of run: blocks into env: maps. Each step now has an env: block with ACTION_PATH (${{ github.action_path }}), and the relevant input variables (ISORT_VERSION, REQUIREMENTS_FILES, CONFIGURATION, SORT_PATHS). Shell commands reference these quoted env vars instead of inline expressions.

2. bin/install_packages: Quoted $isort_version in the pip3 install command (prevents injection via version string). Quoted $file in the pip install -r call (prevents injection via filename). The for-loop over $requirements_files remains unquoted intentionally to allow word-splitting on the space-separated list, which is the intended behavior for multiple files.

3. bin/run_isort: Replaced unquoted $configuration and $sort_paths expansions with bash arrays (IFS=' ' read -ra). This safely splits space-separated arguments into array elements, then passes them as "${config_args[@]}" and "${path_args[@]}" — each element is individually quoted, preventing shell metacharacter injection while preserving multi-argument behavior (e.g., --check-only --diff becomes two separate arguments).

### Iteration 2

**Fixes applied:** invalid-yaml, script-injection

**Notes:**

1. Fixed invalid-yaml in action.yml: Converted all three single-line `run:` values that started with quoted strings to block scalars (`run: |`). YAML was rejecting the trailing unquoted words after the initial quoted scalar. 2. Fixed script-injection in bin/install_packages line 13: Replaced `for file in $requirements_files` (unquoted, susceptible to word-splitting and glob expansion) with `IFS=' ' read -ra files_array <<< "$requirements_files"` followed by `for file in "${files_array[@]}"` to safely iterate over space-separated file paths without shell injection risk.


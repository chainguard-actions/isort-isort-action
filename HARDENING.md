<!-- markdownlint-disable -->

# Hardening Report: isort--isort-action/v0.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **isort--isort-action/v0.1.0** was hardened automatically. 5 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Three `run:` steps in action.yml directly interpolate `${{ ... }}` expressions into shell command strings. Step 1 interpolates `${{ github.action_path }}` directly into the run command. Step 2 interpolates `${{ github.action_path }}`, `${{ inputs.isortVersion }}`, and `${{ inputs.requirementsFiles }}` — the inputs are fully attacker-controlled and are substituted into the shell command before the shell parses it, enabling command injection (e.g. a value like `"; malicious-cmd #` in isortVersion or requirementsFiles). Step 3 similarly interpolates `${{ github.action_path }}`, `${{ inputs.configuration }}`, and `${{ inputs.sortPaths }}`. All three steps must route these values through `env:` variables and reference them as quoted shell variables (e.g. `"$INPUT_VAR"`) instead of using `${{ ... }}` directly inside `run:`.

Locations:

- `action.yml:27`
- `action.yml:29`
- `action.yml:32`

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

Fixed all 5 script injection findings in hardened/action/action.yml. All three run: steps had ${{ }} expressions interpolated directly into shell command strings. Fixed by:
1. Step 1 (ensure_python): Moved ${{ github.action_path }} to env: ACTION_PATH, run command now uses "$ACTION_PATH/bin/ensure_python".
2. Step 2 (install_packages): Moved ${{ github.action_path }}, ${{ inputs.isortVersion }}, and ${{ inputs.requirementsFiles }} to env: block as ACTION_PATH, ISORT_VERSION, REQUIREMENTS_FILES. Run command now uses "$ACTION_PATH/bin/install_packages" "$ISORT_VERSION" "$REQUIREMENTS_FILES".
3. Step 3 (run-isort): Moved ${{ github.action_path }}, ${{ inputs.configuration }}, and ${{ inputs.sortPaths }} to env: block as ACTION_PATH, CONFIGURATION, SORT_PATHS. Run command now uses "$ACTION_PATH/bin/run_isort" "$CONFIGURATION" "$SORT_PATHS". Used quoted "$VAR" (not ${VAR:+"$VAR"}) to preserve original positional argument behavior since the underlying scripts handle empty strings via [ -z ] checks.

### Iteration 2

**Fixes applied:** invalid-yaml, script-injection

**Notes:**

Fixed three findings across three files:

1. action.yml (invalid-yaml): Converted all `run:` fields from invalid single-line quoted form (`run: "cmd" "arg1" "arg2"`) to valid block scalars (`run: |\n  "cmd" "arg1" "arg2"`). This fixes the YAML parse error at line 37.

2. bin/install_packages (script-injection): (a) Quoted `$isort_version` in the pip install command string. (b) Replaced the unquoted `for file in $requirements_files` loop with a `while IFS= read -r file` loop that properly quotes `"$file"` in the pip install call. (c) Single-quoted the static package specifier to prevent bracket glob expansion.

3. bin/run_isort (script-injection): Replaced unquoted `isort $configuration $sort_paths` with bash arrays built via `read -ra` from the double-quoted variables, then expanded as `"${config_args[@]}"` and `"${path_args[@]}"`. This prevents shell metacharacter injection while preserving the ability to pass multiple flags and paths.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted variable `$requirements_files` in `hardened/action/bin/install_packages` line 15. Changed `$(printf '%s\n' $requirements_files)` to `$(printf '%s\n' "$requirements_files")` to prevent word-splitting and glob expansion on the attacker-controlled input value.


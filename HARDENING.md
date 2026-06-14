<!-- markdownlint-disable -->

# Hardening Report: isort--isort-action/v1.1.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **isort--isort-action/v1.1.1** was hardened automatically. 5 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Two `run:` blocks in action.yml directly interpolate `${{ inputs.* }}` expressions into shell command strings before the shell ever sees them. An attacker-controlled calling workflow can supply values containing shell metacharacters (`;`, `|`, `&`, `$(...)`, etc.) to achieve arbitrary command execution.

1. The `install_packages` step (line 73) interpolates `${{ inputs.isort-version || inputs.isortVersion }}` and `${{ inputs.requirements-files || inputs.requirementsFiles }}` directly into the `run:` command.

2. The `run-isort` step (line 81) interpolates `${{ inputs.configuration }}` and `${{ inputs.sort-paths || inputs.sortPaths }}` directly into the `run:` command.

Fix: Move each input into an `env:` variable and reference it as a double-quoted shell variable (e.g., `"$ISORT_VERSION"`) inside the `run:` block.

Locations:

- `action.yml:73`
- `action.yml:81`

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

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Fixed all 5 findings (1 script-injection + 4 static-inline-injection) in action.yml:

1. install_packages step: Moved `${{ inputs.isort-version || inputs.isortVersion }}` to env var `ISORT_VERSION` and `${{ inputs.requirements-files || inputs.requirementsFiles }}` to env var `REQUIREMENTS_FILES`. Used `${REQUIREMENTS_FILES:+"$REQUIREMENTS_FILES"}` so empty requirements-files doesn't pass an empty positional argument.

2. run-isort step: Moved `${{ inputs.configuration }}` to env var `ISORT_CONFIGURATION` and `${{ inputs.sort-paths || inputs.sortPaths }}` to env var `ISORT_SORT_PATHS`. Used unquoted `$ISORT_CONFIGURATION` (with shellcheck disable comment) to allow intentional word-splitting of multi-flag configuration values like `--check-only --diff`. Used `${ISORT_SORT_PATHS:+"$ISORT_SORT_PATHS"}` for the optional sort paths argument.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed the script injection vulnerability in the `run-isort` step of action.yml (line 84). Replaced the unquoted `$ISORT_CONFIGURATION` expansion with a safe bash array approach: `read -ra config_flags <<< "$ISORT_CONFIGURATION"` followed by `"${config_flags[@]}"`. This performs word-splitting on the configuration flags without allowing shell metacharacters to be interpreted as shell commands. The `# shellcheck disable=SC2086` comment was removed since it's no longer needed.


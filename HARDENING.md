<!-- markdownlint-disable -->

# Hardening Report: isort--isort-action/v1.1.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **isort--isort-action/v1.1.1** was hardened automatically. 7 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): `${{ inputs.* }}` expressions are directly interpolated inside `run:` shell command strings in action.yml. In the install_packages step, `${{ inputs.isort-version || inputs.isortVersion }}` and `${{ inputs.requirements-files || inputs.requirementsFiles }}` are passed unquoted directly to the shell, allowing an attacker-controlled input to inject shell metacharacters. In the run-isort step, `${{ inputs.configuration }}` and `${{ inputs.sort-paths || inputs.sortPaths }}` are similarly interpolated. These values are substituted by the Actions runner before the shell ever sees them, bypassing any shell quoting. The safe fix is to pass inputs via `env:` variables and reference them as double-quoted shell variables (e.g. `"$INPUT_CONFIGURATION"`).

Sub-rule (a) also applies in test.yaml: `${{steps.imports-properly-sorted.outputs.isort-result}}` and `${{steps.imports-improperly-sorted.outputs.isort-result}}` are interpolated directly inside `run:` blocks. Even though the assignment uses double quotes, the expression is expanded by the Actions runner before the shell parses the script, so any newlines or shell metacharacters in the isort output can break out of the quoted string.

Locations:

- `action.yml:73`
- `action.yml:74`
- `action.yml:81`
- `action.yml:82`
- `.github/workflows/test.yaml:22`
- `.github/workflows/test.yaml:31`

### unpinned-uses (severity: high)

Both workflow files reference `actions/checkout@v4`, which is a mutable tag rather than a pinned 40-character commit SHA. A tag can be moved to point to a different (potentially malicious) commit at any time, creating a supply-chain risk. Each reference should be replaced with the full SHA, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `.github/workflows/lint.yaml:9`
- `.github/workflows/test.yaml:12`

### missing-permissions (severity: medium)

Neither workflow file defines a `permissions:` block at the top level or at the job level. Without explicit permissions, GitHub Actions grants the default token permissions (which may include `write` access to contents and other scopes depending on repository settings). Both `lint.yaml` and `test.yaml` should declare minimal required permissions, e.g. `permissions: read-all` or specific scopes such as `contents: read`.

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

Fixed all findings in action.yml and both workflow files:

1. action.yml (script-injection/static-inline-injection): Moved all ${{ inputs.* }} expressions to env: blocks. For install_packages step: INPUT_ISORT_VERSION and INPUT_REQUIREMENTS_FILES are passed as double-quoted shell variables. For run-isort step: INPUT_CONFIGURATION and INPUT_SORT_PATHS are tokenized via xargs into bash arrays (since they are whitespace-separated argument lists), then expanded safely with "${cfg_args[@]}" and "${path_args[@]}".

2. test.yaml (script-injection): Moved ${{steps.*.outputs.isort-result}} expressions into env: blocks as ISORT_RESULT, then referenced as "$ISORT_RESULT" in the shell scripts.

3. lint.yaml and test.yaml (unpinned-uses): Pinned actions/checkout@v4 to full commit SHA 11d5960a326750d5838078e36cf38b85af677262 with # v4 comment.

4. lint.yaml and test.yaml (missing-permissions): Added 'permissions: contents: read' at the workflow top level in both files.


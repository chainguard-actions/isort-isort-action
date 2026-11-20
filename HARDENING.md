<!-- markdownlint-disable -->

# Hardening Report: isort--isort-action/v1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **isort--isort-action/v1** was hardened automatically. 8 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): action inputs are placed straight into a run: script with ${{ }} expressions in the 'install_packages' step: `$GITHUB_ACTION_PATH/bin/install_packages ${{ inputs.isort-version || inputs.isortVersion }} ${{ inputs.requirements-files || inputs.requirementsFiles }}`. Shell metacharacters in these inputs are run as commands. Fix: pass the inputs through env: variables and reference them as quoted shell variables.

Locations:

- `action.yml:82`

### script-injection (severity: high)

Rule (a): action inputs are placed straight into a run: script with ${{ }} expressions in the 'run-isort' step: `$GITHUB_ACTION_PATH/bin/run_isort ${{ inputs.configuration }} ${{ inputs.sort-paths || inputs.sortPaths }}`. Shell metacharacters in these inputs are run as commands. Fix: pass the inputs through env: variables. If a value is a whitespace-separated list, load it into an array (with a shellcheck SC2206 disable directive) or quote it.

Locations:

- `action.yml:90`

### github-env-injection (severity: high)

bin/run_isort (called from the run-isort step) writes isort output to $GITHUB_OUTPUT as a multi-line heredoc. The output comes from untrusted inputs and repository file contents/diffs. The delimiter is not cryptographically random: awk's srand() with no argument seeds from the current time, so the 8-character delimiter can be predicted. Content containing a line equal to the delimiter can end the block early and inject extra outputs. Fix: build the delimiter from /dev/urandom, openssl rand or uuidgen.

Locations:

- `bin/run_isort:15`
- `bin/run_isort:28`

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

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index 13ffa3c..6200b9a 100644
--- a/action.yml
+++ b/action.yml
@@ -4,7 +4,6 @@ description: Run isort on a Python project
 branding:
   icon: check
   color: blue
-
 inputs:
   isort-version:
     description: Version of isort to use
@@ -15,13 +14,12 @@ inputs:
     #
     # TODO: Change to latest when dropping support for isortVersion.
     default: null
-
   isortVersion:
     description: >
       Version of isort to use (DEPRECATED: use isort-version instead)
+
     required: false
     default: latest
-
   sort-paths:
     description: files or directories to sort
     required: false
@@ -31,22 +29,20 @@ inputs:
     #
     # TODO: Change to . when dropping support for sortPaths.
     default: null
-
   sortPaths:
     description: >
       files or directories to sort (DEPRECATED: use sort-paths instead)
+
     required: false
     default: .
-
   configuration:
     description: isort configuration options
     required: false
     default: --check-only --diff
-
   requirements-files:
     description: >
-      path(s) to requirements files that should be installed to properly
-      configure third-party imports
+      path(s) to requirements files that should be installed to properly configure third-party imports
+
     required: false
     # Use default null to fallback to requirementsFiles with the expression:
     #
@@ -54,39 +50,56 @@ inputs:
     #
     # TODO: Change to "" when dropping support for requirementsFiles.
     default: null
-
   requirementsFiles:
     description: >
-      path(s) to requirements files that should be installed to properly
-      configure third-party imports (DEPRECATED: use requirement-files instead)
+      path(s) to requirements files that should be installed to properly configure third-party imports (DEPRECATED: use requirement-files instead)
+
     required: false
     default: ""
-
 outputs:
   isort-result:
     description: isort result
     value: ${{ steps.run-isort.outputs.isort-output }}
-
 runs:
   using: composite
   steps:
     - run: $GITHUB_ACTION_PATH/bin/ensure_python
       shell: bash
     - run: >
         python -m venv "${HOME}/isort-venv"
+
       shell: bash
-    - run: >
-        $GITHUB_ACTION_PATH/bin/install_packages
-        ${{ inputs.isort-version || inputs.isortVersion }}
-        ${{ inputs.requirements-files || inputs.requirementsFiles }}
+    - env:
+        ISORT_VERSION: ${{ inputs.isort-version || inputs.isortVersion }}
+        REQUIREMENTS_FILES: ${{ inputs.requirements-files || inputs.requirementsFiles }}
+      run: "$GITHUB_ACTION_PATH/bin/install_packages \"$ISORT_VERSION\" \"$REQUIREMENTS_FILES\""
       shell: bash
     - run: echo "::add-matcher::$GITHUB_ACTION_PATH/isort-matcher.json"
       shell: bash
     - id: run-isort
-      run: >
-        $GITHUB_ACTION_PATH/bin/run_isort
-        ${{ inputs.configuration }}
-        ${{ inputs.sort-paths || inputs.sortPaths }}
+      env:
+        ISORT_CONFIGURATION: ${{ inputs.configuration }}
+        ISORT_SORT_PATHS: ${{ inputs.sort-paths || inputs.sortPaths }}
+      run: |
+        read -ra config_args <<< "$ISORT_CONFIGURATION"
+        "$GITHUB_ACTION_PATH/bin/run_isort" "${config_args[@]}" "$ISORT_SORT_PATHS"
       shell: bash
     - run: echo "::remove-matcher owner=isort-matcher::"
       shell: bash

```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, github-env-injection, prior-fix

**Notes:**

action.yml: I moved all four inputs out of the run: scripts into env: (ISORT_VERSION, REQUIREMENTS_FILES, ISORT_CONFIGURATION, ISORT_SORT_PATHS). Upstream pasted these inputs straight into the script, so the shell split them on whitespace and honoured quotes. The new code splits each non-empty value the same quote-aware way: an 'xargs -r true' check rejects an unmatched quote, then the tokens are read into an array through xargs and the shell's built-in printf. The bin scripts then get the same argv as upstream (for example, '--check-only --diff' arrives as two separate arguments). This replaces the previous harden's approach, which passed requirements files and sort paths as a single quoted argument and broke multi-value inputs. One remaining difference: upstream's unquoted inline values were also expanded as glob patterns, and the xargs split does not do that. A pattern such as 'src/*.py' in sort-paths now reaches isort as literal text instead of a list of matching files. I followed the prescribed split for inline inputs, which keeps the quote handling but drops the globbing; plain paths and flags are not affected. bin/run_isort: the multiline GITHUB_OUTPUT delimiter came from a time-seeded awk srand(), so it could be predicted. It is now built from /dev/urandom (EOF_ plus 32 hex characters), and the output value and its lines are written unchanged. Nothing was run or tested.


<!-- markdownlint-disable -->

# Hardening Report: isort--isort-action/v1.1.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **isort--isort-action/v1.1.1** was hardened automatically. 8 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): the install_packages step puts GitHub expressions straight into the run: script: `${{ inputs.isort-version || inputs.isortVersion }}` and `${{ inputs.requirements-files || inputs.requirementsFiles }}`. A caller can inject shell metacharacters through these inputs. Fix: pass the values through env: variables and expand them quoted, or in guarded form for optional arguments.

Locations:

- `action.yml:80`

### script-injection (severity: high)

Rule (a): the run-isort step puts GitHub expressions straight into the run: script: `${{ inputs.configuration }}` and `${{ inputs.sort-paths || inputs.sortPaths }}`. A caller can inject shell metacharacters through these inputs. Fix: pass the values through env: variables. If they are whitespace-separated lists, expand them into an array with an SC2206 shellcheck directive.

Locations:

- `action.yml:86`

### github-env-injection (severity: high)

bin/run_isort is called by the run-isort step. It writes isort output to $GITHUB_OUTPUT between heredoc delimiters. That output comes from the repository's files and isort arguments, so an attacker can influence it. The delimiter comes from awk's srand()/rand(), and srand() is seeded from the current time in seconds. That is not a cryptographically secure random source, so the delimiter can be guessed. A guessed delimiter line in the output would end the block early and inject extra output entries. Fix: build the delimiter from /dev/urandom, openssl rand or uuidgen.

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
index 13ffa3c..7e10733 100644
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
@@ -54,39 +50,66 @@ inputs:
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
+        INPUT_ISORT_VERSION: ${{ inputs.isort-version || inputs.isortVersion }}
+        INPUT_REQUIREMENTS_FILES: ${{ inputs.requirements-files || inputs.requirementsFiles }}
+      run: |
+        "$GITHUB_ACTION_PATH/bin/install_packages" "$INPUT_ISORT_VERSION" "$INPUT_REQUIREMENTS_FILES"
       shell: bash
     - run: echo "::add-matcher::$GITHUB_ACTION_PATH/isort-matcher.json"
       shell: bash
     - id: run-isort
-      run: >
-        $GITHUB_ACTION_PATH/bin/run_isort
-        ${{ inputs.configuration }}
-        ${{ inputs.sort-paths || inputs.sortPaths }}
+      env:
+        INPUT_CONFIGURATION: ${{ inputs.configuration }}
+        INPUT_SORT_PATHS: ${{ inputs.sort-paths || inputs.sortPaths }}
+      run: |
+        cfg_args=()
+        if [ -n "$INPUT_CONFIGURATION" ]; then
+          while IFS= read -r -d '' t; do cfg_args+=("$t"); done \
+            < <(printf '%s' "$INPUT_CONFIGURATION" | xargs printf '%s\0')
+        fi
+        path_args=()
+        if [ -n "$INPUT_SORT_PATHS" ]; then
+          while IFS= read -r -d '' t; do path_args+=("$t"); done \
+            < <(printf '%s' "$INPUT_SORT_PATHS" | xargs printf '%s\0')
+        fi
+        "$GITHUB_ACTION_PATH/bin/run_isort" "${cfg_args[@]}" "${path_args[@]}"
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

I moved every input expression in the install_packages and run-isort steps of action.yml into env: variables. Upstream pasted these inputs straight into the script, so they used to be split the way the shell splits text, quotes included. To keep that splitting, the requirements files, configuration and sort paths lists are now split with the quote-aware xargs method (with checks for empty values and unmatched quotes) and passed as bash arrays. The isort version is passed with ${VAR:+"$VAR"}, so an empty value adds no argument. bin/run_isort used a guessable delimiter built from awk's time-seeded srand()/rand(); it now builds the delimiter from /dev/urandom. The lines of the multi-line isort output are kept as they are. These changes also cover the earlier harden's action.yml fixes (the prior-fix finding), with a quote-aware xargs split in place of the old xargs printf one. Nothing was run or tested here.


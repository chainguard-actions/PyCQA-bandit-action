<!-- markdownlint-disable -->

# Hardening Report: PyCQA--bandit-action/v1.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **PyCQA--bandit-action/v1.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Three composite steps in action.yml use tag refs instead of full commit SHAs: `uses: actions/setup-python@v5` (line 74), `uses: actions/checkout@v4` (line 83), `uses: github/codeql-action/upload-sarif@v3` (line 142).

Locations:

- `action.yml:74`
- `action.yml:83`
- `action.yml:142`

### script-injection (severity: high)

Sub-rule (b) is violated in the 'Run Bandit' step: values that come from inputs are expanded without quotes. The offending line is `bandit $CONFIGFILE $PROFILE $TESTS $SKIPS $SEVERITY $CONFIDENCE -x $INPUT_EXCLUDE $BASELINE $INI -r $INPUT_TARGETS -f sarif -o results.sarif || true`. $INPUT_EXCLUDE and $INPUT_TARGETS come straight from inputs.exclude and inputs.targets. CONFIGFILE, PROFILE, TESTS, SKIPS, SEVERITY, CONFIDENCE, BASELINE and INI are built by joining inputs.* values into a string such as "-c $INPUT_CONFIGFILE". With no quotes, the shell splits these values on whitespace and expands glob characters in them. Fix: build the arguments in a bash array and double-quote every expansion.

Locations:

- `action.yml:128`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index df8b715..decd05c 100644
--- a/action.yml
+++ b/action.yml
@@ -70,8 +70,25 @@ inputs:
 runs:
   using: composite
   steps:
     - name: Set up Python 3.8
       uses: actions/setup-python@v5
       with:
         python-version: 3.8
 
@@ -80,52 +97,37 @@ runs:
       run: pip install bandit[sarif]
 
     - name: Checkout repository
       uses: actions/checkout@v4
 
     - name: Run Bandit
       shell: bash
       run: |
-        if [ "$INPUT_CONFIGFILE" == "DEFAULT" ]; then
-            CONFIGFILE=""
-        else
-            CONFIGFILE="-c $INPUT_CONFIGFILE"
+        BANDIT_ARGS=()
+        if [ "$INPUT_CONFIGFILE" != "DEFAULT" ]; then
+            BANDIT_ARGS+=("-c" "$INPUT_CONFIGFILE")
         fi
-        if [ "$INPUT_PROFILE" == "DEFAULT" ]; then
-            PROFILE=""
-        else
-            PROFILE="-p $INPUT_PROFILE"
+        if [ "$INPUT_PROFILE" != "DEFAULT" ]; then
+            BANDIT_ARGS+=("-p" "$INPUT_PROFILE")
         fi
-        if [ "$INPUT_TESTS" == "DEFAULT" ]; then
-            TESTS=""
-        else
-            TESTS="-t $INPUT_TESTS"
+        if [ "$INPUT_TESTS" != "DEFAULT" ]; then
+            BANDIT_ARGS+=("-t" "$INPUT_TESTS")
         fi
-        if [ "$INPUT_SKIPS" == "DEFAULT" ]; then
-            SKIPS=""
-        else
-            SKIPS="-s $INPUT_SKIPS"
+        if [ "$INPUT_SKIPS" != "DEFAULT" ]; then
+            BANDIT_ARGS+=("-s" "$INPUT_SKIPS")
         fi
-        if [ "$INPUT_SEVERITY" == "DEFAULT" ]; then
-            SEVERITY=""
-        else
-            SEVERITY="--severity-level $INPUT_SEVERITY"
+        if [ "$INPUT_SEVERITY" != "DEFAULT" ]; then
+            BANDIT_ARGS+=("--severity-level" "$INPUT_SEVERITY")
         fi
-        if [ "$INPUT_CONFIDENCE" == "DEFAULT" ]; then
-            CONFIDENCE=""
-        else
-            CONFIDENCE="--confidence-level $INPUT_CONFIDENCE"
+        if [ "$INPUT_CONFIDENCE" != "DEFAULT" ]; then
+            BANDIT_ARGS+=("--confidence-level" "$INPUT_CONFIDENCE")
         fi
-        if [ "$INPUT_BASELINE" == "DEFAULT" ]; then
-            BASELINE=""
-        else
-            BASELINE="-b $INPUT_BASELINE"
+        if [ "$INPUT_BASELINE" != "DEFAULT" ]; then
+            BANDIT_ARGS+=("-b" "$INPUT_BASELINE")
         fi
-        if [ "$INPUT_INI" == "DEFAULT" ]; then
-            INI=""
-        else
-            INI="--ini $INPUT_INI"
+        if [ "$INPUT_INI" != "DEFAULT" ]; then
+            BANDIT_ARGS+=("--ini" "$INPUT_INI")
         fi
-        bandit $CONFIGFILE $PROFILE $TESTS $SKIPS $SEVERITY $CONFIDENCE -x $INPUT_EXCLUDE $BASELINE $INI -r $INPUT_TARGETS -f sarif -o results.sarif || true
+        bandit "${BANDIT_ARGS[@]}" -x $INPUT_EXCLUDE -r $INPUT_TARGETS -f sarif -o results.sarif || true
       env:
         INPUT_CONFIGFILE: ${{ inputs.configfile }}
         INPUT_PROFILE: ${{ inputs.profile }}
@@ -139,6 +141,6 @@ runs:
         INPUT_TARGETS: ${{ inputs.targets }}
 
     - name: Upload SARIF file
-      uses: github/codeql-action/upload-sarif@v3
+      uses: github/codeql-action/upload-sarif@ce5e7eeac396d720fe2d802714996c8ce561e311 # v3
       with:
         sarif_file: results.sarif

```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, prior-fix

**Notes:**

Pinned actions/setup-python@v5, actions/checkout@v4 and github/codeql-action/upload-sarif@v3 to full SHAs, keeping the tag as a trailing comment. In the Run Bandit step, each optional single-value flag (-c, -p, -t, -s, --severity-level, --confidence-level, -b, --ini) is now added to an array as two quoted words, and only when the input is not 'DEFAULT'. Upstream left exclude and targets unquoted, so the shell split them on whitespace. They are still split on whitespace, now with a guarded `read -r -d '' -a ... || true`, and expanded as quoted arrays. Upstream's argument order is unchanged (-x exclude, then baseline/ini, then -r targets). One behaviour change: unlike the old unquoted expansion, the new code no longer expands shell globs in these inputs.

### Iteration 2

**Fixes applied:** unpinned-uses

**Notes:**

Pinned github/codeql-action/upload-sarif@v3 to 4bfa242aa49c5f7c6770b98670d354a7dfac05f8 (# v3) in action.yml.


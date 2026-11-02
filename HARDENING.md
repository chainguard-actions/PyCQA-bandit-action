<!-- markdownlint-disable -->

# Hardening Report: PyCQA--bandit-action/v1.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **PyCQA--bandit-action/v1.0.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Three composite action steps in action.yml use tags instead of full commit SHAs: 'uses: actions/setup-python@v5' (line 74), 'uses: actions/checkout@v4' (line 83) and 'uses: github/codeql-action/upload-sarif@v3' (line 142).

Locations:

- `action.yml:74`
- `action.yml:83`
- `action.yml:142`

### script-injection (severity: high)

Sub-rule (b): in the 'Run Bandit' step, env vars that come from inputs are expanded without quotes. The line is: 'bandit $CONFIGFILE $PROFILE $TESTS $SKIPS $SEVERITY $CONFIDENCE -x $INPUT_EXCLUDE $BASELINE $INI -r $INPUT_TARGETS -f sarif -o results.sarif || true'. $INPUT_EXCLUDE and $INPUT_TARGETS come straight from inputs.exclude and inputs.targets. CONFIGFILE, PROFILE, TESTS, SKIPS, SEVERITY, CONFIDENCE, BASELINE and INI hold inputs text joined onto flags, for example CONFIGFILE="-c $INPUT_CONFIGFILE". With no quotes, the shell splits these values on spaces and expands glob characters, so input values can add extra arguments or patterns. They should be quoted, or the command should be built as a bash array with each argument quoted.

Locations:

- `action.yml:128`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index 7be6f1c..46b22ae 100644
--- a/action.yml
+++ b/action.yml
@@ -70,8 +70,25 @@ inputs:
 runs:
   using: composite
   steps:
     - name: Set up Python 3.9
       uses: actions/setup-python@v5
       with:
         python-version: 3.9
 
@@ -80,52 +97,37 @@ runs:
       run: pip install bandit[sarif,toml]
 
     - name: Checkout repository
       uses: actions/checkout@v4
 
     - name: Run Bandit
       shell: bash
       run: |
-        if [ "$INPUT_CONFIGFILE" == "DEFAULT" ]; then
-            CONFIGFILE=""
-        else
-            CONFIGFILE="-c $INPUT_CONFIGFILE"
+        args=()
+        if [ "$INPUT_CONFIGFILE" != "DEFAULT" ]; then
+            args+=(-c "$INPUT_CONFIGFILE")
         fi
-        if [ "$INPUT_PROFILE" == "DEFAULT" ]; then
-            PROFILE=""
-        else
-            PROFILE="-p $INPUT_PROFILE"
+        if [ "$INPUT_PROFILE" != "DEFAULT" ]; then
+            args+=(-p "$INPUT_PROFILE")
         fi
-        if [ "$INPUT_TESTS" == "DEFAULT" ]; then
-            TESTS=""
-        else
-            TESTS="-t $INPUT_TESTS"
+        if [ "$INPUT_TESTS" != "DEFAULT" ]; then
+            args+=(-t "$INPUT_TESTS")
         fi
-        if [ "$INPUT_SKIPS" == "DEFAULT" ]; then
-            SKIPS=""
-        else
-            SKIPS="-s $INPUT_SKIPS"
+        if [ "$INPUT_SKIPS" != "DEFAULT" ]; then
+            args+=(-s "$INPUT_SKIPS")
         fi
-        if [ "$INPUT_SEVERITY" == "DEFAULT" ]; then
-            SEVERITY=""
-        else
-            SEVERITY="--severity-level $INPUT_SEVERITY"
+        if [ "$INPUT_SEVERITY" != "DEFAULT" ]; then
+            args+=(--severity-level "$INPUT_SEVERITY")
         fi
-        if [ "$INPUT_CONFIDENCE" == "DEFAULT" ]; then
-            CONFIDENCE=""
-        else
-            CONFIDENCE="--confidence-level $INPUT_CONFIDENCE"
+        if [ "$INPUT_CONFIDENCE" != "DEFAULT" ]; then
+            args+=(--confidence-level "$INPUT_CONFIDENCE")
         fi
-        if [ "$INPUT_BASELINE" == "DEFAULT" ]; then
-            BASELINE=""
-        else
-            BASELINE="-b $INPUT_BASELINE"
+        if [ "$INPUT_BASELINE" != "DEFAULT" ]; then
+            args+=(-b "$INPUT_BASELINE")
         fi
-        if [ "$INPUT_INI" == "DEFAULT" ]; then
-            INI=""
-        else
-            INI="--ini $INPUT_INI"
+        if [ "$INPUT_INI" != "DEFAULT" ]; then
+            args+=(--ini "$INPUT_INI")
         fi
-        bandit $CONFIGFILE $PROFILE $TESTS $SKIPS $SEVERITY $CONFIDENCE -x $INPUT_EXCLUDE $BASELINE $INI -r $INPUT_TARGETS -f sarif -o results.sarif || true
+        bandit "${args[@]}" -x $INPUT_EXCLUDE -r $INPUT_TARGETS -f sarif -o results.sarif || true
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

I pinned three actions to full commit SHAs: setup-python@v5 to a26af69b, checkout@v4 to 11d5960a and codeql-action/upload-sarif@v3 to 4bfa242a. In the 'Run Bandit' step, each optional flag is now added to a bash array as two separate words, the flag and its quoted value. -x "$INPUT_EXCLUDE" is passed as one quoted value because bandit's -x takes a single comma-separated list. Targets can be several paths, so the input is split on whitespace into an array with read -r -d '' -a (|| true), which keeps upstream's splitting without globbing or running anything. This also re-applies the earlier harden's security changes (the prior-fix finding).


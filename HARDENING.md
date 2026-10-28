<!-- markdownlint-disable -->

# Hardening Report: PyCQA--bandit-action/v1.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **PyCQA--bandit-action/v1.0.1** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Composite action steps reference actions by mutable tags rather than full 40-char commit SHAs: `uses: actions/setup-python@v5` (line 74), `uses: actions/checkout@v4` (line 83), `uses: github/codeql-action/upload-sarif@v3` (line 142).

Locations:

- `action.yml:74`
- `action.yml:83`
- `action.yml:142`

### script-injection (severity: high)

Sub-rule (b) violated in the 'Run Bandit' step: workflow-controlled input env vars are expanded without quotes. Offending line: `bandit $CONFIGFILE $PROFILE $TESTS $SKIPS $SEVERITY $CONFIDENCE -x $INPUT_EXCLUDE $BASELINE $INI -r $INPUT_TARGETS -f sarif -o results.sarif || true`. $INPUT_EXCLUDE and $INPUT_TARGETS (from inputs.exclude and inputs.targets) are unquoted. $CONFIGFILE, $PROFILE, $TESTS, $SKIPS, $SEVERITY, $CONFIDENCE, $BASELINE and $INI are built from inputs.* values (e.g. CONFIGFILE="-c $INPUT_CONFIGFILE") and are also expanded unquoted. Because of this, word splitting and glob expansion apply to input data. Pass each value as its own quoted argument, using the guarded form (e.g. ${INPUT_CONFIGFILE:+-c "$INPUT_CONFIGFILE"}) or a bash array.

Locations:

- `action.yml:128`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Pinned setup-python, checkout and codeql-action/upload-sarif to full commit SHAs, keeping the tag as a trailing comment. The Run Bandit step now builds the bandit command as a bash array. Each optional flag goes in as two separate items, the flag and its quoted value, and only when the input isn't 'DEFAULT'. That includes -c, -p, -t, -s, --severity-level, --confidence-level, -b and --ini. The exclude value is passed as a single quoted argument, so a pattern like *.egg goes straight to bandit and the shell no longer expands it. targets can hold several paths, so it is split on whitespace with read -r -d '' -a, which matches how upstream's unquoted $INPUT_TARGETS was split. That split is guarded so an empty value adds no argument. One behaviour change: targets are no longer glob-expanded by the shell, so a target like src/*.py is now passed to bandit as literal text. I haven't run the action to test these changes.

### Iteration 2

**Fixes applied:** unpinned-uses

**Notes:**

Fixed the broken last step of action.yml. It is now named 'Upload SARIF' and has its own key: `uses: github/codeql-action/upload-sarif@4bfa242aa49c5f7c6770b98670d354a7dfac05f8 # v3`. The SHA was checked with lookup_action_sha.


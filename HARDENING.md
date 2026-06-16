<!-- markdownlint-disable -->

# Hardening Report: PyCQA--bandit-action/v1.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **PyCQA--bandit-action/v1.0.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Three `uses:` references in action.yml use mutable version tags instead of pinned 40-character SHA digests, making the action vulnerable to supply-chain attacks if the upstream tag is moved or compromised:
- `actions/setup-python@v5` (line 64)
- `actions/checkout@v4` (line 69)
- `github/codeql-action/upload-sarif@v3` (line 119)
Each should be pinned to a full commit SHA, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `action.yml:64`
- `action.yml:69`
- `action.yml:119`

### script-injection (severity: high)

Sub-rule (b) violation: The 'Run Bandit' step maps all `inputs.*` values into env vars (e.g. `INPUT_CONFIGFILE: ${{ inputs.configfile }}`) and then expands those env vars **unquoted** throughout the shell script. Unquoted expansions allow shell metacharacters (`;`, `|`, `&`, `$(...)`, whitespace, glob chars) embedded in user-supplied input to be interpreted by the shell.

Specific unquoted expansions in intermediate assignments:
  `CONFIGFILE="-c $INPUT_CONFIGFILE"` — $INPUT_CONFIGFILE unquoted inside double-quotes but then $CONFIGFILE itself is unquoted at use
  `PROFILE="-p $INPUT_PROFILE"`, `TESTS="-t $INPUT_TESTS"`, `SKIPS="-s $INPUT_SKIPS"`, etc.

Final command line (line 107) uses all variables unquoted:
  `bandit $CONFIGFILE $PROFILE $TESTS $SKIPS $SEVERITY $CONFIDENCE -x $INPUT_EXCLUDE $BASELINE $INI -r $INPUT_TARGETS -f sarif -o results.sarif || true`

All of `$CONFIGFILE`, `$PROFILE`, `$TESTS`, `$SKIPS`, `$SEVERITY`, `$CONFIDENCE`, `$INPUT_EXCLUDE`, `$BASELINE`, `$INI`, and `$INPUT_TARGETS` are unquoted and hold user-controlled input values.

Locations:

- `action.yml:107`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed three unpinned `uses:` references by pinning them to full SHA digests: actions/setup-python@v5 → @a26af69be951a213d495a4c3e4e4022e16d87065, actions/checkout@v4 → @34e114876b0b11c390a56381ad16ebd13914f8d5, github/codeql-action/upload-sarif@v3 → @dd903d2e4f5405488e5ef1422510ee31c8b32357. Fixed script-injection by replacing the unquoted variable expansions in the bandit command with a bash array (BANDIT_ARGS) where each flag and user-supplied value is added as a separately double-quoted array element, then invoked as `bandit "${BANDIT_ARGS[@]}"`. This ensures shell metacharacters in user-controlled inputs cannot be interpreted by the shell.


<!-- markdownlint-disable -->

# Hardening Report: PyCQA--bandit-action/v1.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **PyCQA--bandit-action/v1.0.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Three `uses:` references in action.yml use mutable version tags instead of pinned 40-character SHA commit hashes, making the action vulnerable to supply-chain attacks if those tags are moved:
- `uses: actions/setup-python@v5` (line 72)
- `uses: actions/checkout@v4` (line 78)
- `uses: github/codeql-action/upload-sarif@v3` (line 127)
Each should be pinned to a full SHA, e.g. `actions/setup-python@<40-hex-sha> # v5`.

Locations:

- `action.yml:72`
- `action.yml:78`
- `action.yml:127`

### script-injection (severity: high)

Rule (b) violation: The 'Run Bandit' step constructs intermediate shell variables (`CONFIGFILE`, `PROFILE`, `TESTS`, `SKIPS`, `SEVERITY`, `CONFIDENCE`, `BASELINE`, `INI`) by embedding `$INPUT_*` env vars (sourced from `inputs.*`) without quoting, and then passes all of them unquoted to the `bandit` command:
  `bandit $CONFIGFILE $PROFILE $TESTS $SKIPS $SEVERITY $CONFIDENCE -x $INPUT_EXCLUDE $BASELINE $INI -r $INPUT_TARGETS ...`
Because the shell expands these variables without double-quotes, an attacker-controlled input value containing shell metacharacters (`;`, `|`, `&`, `$(...)`, etc.) can break out of the intended argument context and execute arbitrary commands. All of `$CONFIGFILE`, `$PROFILE`, `$TESTS`, `$SKIPS`, `$SEVERITY`, `$CONFIDENCE`, `$INPUT_EXCLUDE`, `$BASELINE`, `$INI`, and `$INPUT_TARGETS` must be double-quoted at the point of use.

Locations:

- `action.yml:113`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all three unpinned `uses:` references by pinning them to their full 40-character SHA hashes (with the original tag preserved as a comment). Fixed the script-injection vulnerability in the 'Run Bandit' step by replacing string variables that packed flag+value pairs (which required unquoted expansion) with a bash array. Each optional flag and its value are now appended as separate quoted array elements, and the final bandit invocation uses `"${args[@]}"` for safe expansion. `$INPUT_EXCLUDE` and `$INPUT_TARGETS` are also double-quoted at the point of use.


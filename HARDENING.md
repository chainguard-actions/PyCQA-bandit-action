<!-- markdownlint-disable -->

# Hardening Report: PyCQA--bandit-action/v1.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **PyCQA--bandit-action/v1.0.1** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Three `uses:` references in action.yml use mutable tags instead of pinned full 40-character SHA digests, making the action vulnerable to supply-chain attacks if the referenced tag is moved or overwritten:
- `uses: actions/setup-python@v5` (line 74)
- `uses: actions/checkout@v4` (line 83)
- `uses: github/codeql-action/upload-sarif@v3` (line 121)
Each should be pinned to a full commit SHA, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `action.yml:74`
- `action.yml:83`
- `action.yml:121`

### script-injection (severity: high)

Sub-rule (b) violation: The 'Run Bandit' step maps all `inputs.*` values into env vars (INPUT_CONFIGFILE, INPUT_PROFILE, INPUT_TESTS, INPUT_SKIPS, INPUT_SEVERITY, INPUT_CONFIDENCE, INPUT_EXCLUDE, INPUT_BASELINE, INPUT_INI, INPUT_TARGETS) via the `env:` block, but then expands them unquoted inside the `run:` shell script. Specifically:
1. Intermediate variables are built with unquoted expansions, e.g. `CONFIGFILE="-c $INPUT_CONFIGFILE"` (line 91) — the value of INPUT_CONFIGFILE is not quoted, allowing glob expansion.
2. The final `bandit` invocation (line 108) uses all intermediate variables and two direct env vars completely unquoted: `bandit $CONFIGFILE $PROFILE $TESTS $SKIPS $SEVERITY $CONFIDENCE -x $INPUT_EXCLUDE $BASELINE $INI -r $INPUT_TARGETS`. An attacker-controlled input containing shell metacharacters (`;`, `|`, `&`, `$(...)`, whitespace, globs) can break out of the intended argument context and inject arbitrary shell commands. All these variables should be double-quoted: `"$CONFIGFILE"`, `"$INPUT_EXCLUDE"`, `"$INPUT_TARGETS"`, etc.

Locations:

- `action.yml:91`
- `action.yml:108`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed three unpinned uses: references by pinning to full commit SHAs (actions/setup-python@a26af69be951a213d495a4c3e4e4022e16d87065, actions/checkout@11d5960a326750d5838078e36cf38b85af677262, github/codeql-action/upload-sarif@ce5e7eeac396d720fe2d802714996c8ce561e311). Fixed script injection in the 'Run Bandit' step by replacing the pattern of building unquoted flag+value strings with a bash array approach: each optional flag and its value are appended to an `args=()` array as separate properly-quoted elements, and the final bandit invocation uses `"${args[@]}"` along with double-quoted `"$INPUT_EXCLUDE"` and `"$INPUT_TARGETS"` to prevent shell metacharacter injection.

### Iteration 1

**Fixes applied:** publish-gate

**Notes:**

Fixed action.yml line 113: changed `bandit "${args[@]}" -x "$INPUT_EXCLUDE" -r "$INPUT_TARGETS"` to `bandit "${args[@]}" -x $INPUT_EXCLUDE -r $INPUT_TARGETS`. The double-quoted variables were passing the entire exclude/targets values as single arguments, but upstream intended them to be word-split by the shell into multiple arguments (e.g., space-separated paths). Unquoted env variable expansion is safe here — it only word-splits and globs, it does not evaluate shell metacharacters like semicolons, pipes, or command substitutions.


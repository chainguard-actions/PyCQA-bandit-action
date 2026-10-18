<!-- markdownlint-disable -->

# Hardening Report: PyCQA--bandit-action/v1.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **PyCQA--bandit-action/v1.0.0** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Three `uses:` references in action.yml use mutable tag refs instead of pinned 40-character SHA digests, making the action vulnerable to supply-chain attacks if those tags are moved:
- `actions/setup-python@v5` (line 68)
- `actions/checkout@v4` (line 74)
- `github/codeql-action/upload-sarif@v3` (line 124)
Each should be replaced with a full commit SHA, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `action.yml:68`
- `action.yml:74`
- `action.yml:124`

### script-injection (severity: high)

Rule (b) violation: The 'Run Bandit' step's `run:` block expands env vars that hold `inputs.*` values without double-quoting them, allowing shell metacharacter injection by an attacker who controls those inputs.

Inside the `else` branches, variables are assigned without quoting the source:
  `CONFIGFILE="-c $INPUT_CONFIGFILE"` (line 84) — $INPUT_CONFIGFILE unquoted
  `PROFILE="-p $INPUT_PROFILE"` (line 88) — unquoted
  `TESTS="-t $INPUT_TESTS"` (line 92) — unquoted
  `SKIPS="-s $INPUT_SKIPS"` (line 96) — unquoted
  `SEVERITY="--severity-level $INPUT_SEVERITY"` (line 100) — unquoted
  `CONFIDENCE="--confidence-level $INPUT_CONFIDENCE"` (line 104) — unquoted
  `BASELINE="-b $INPUT_BASELINE"` (line 108) — unquoted
  `INI="--ini $INPUT_INI"` (line 112) — unquoted

The final bandit invocation (line 113) also expands all assembled variables and two INPUT_* vars without quoting:
  `bandit $CONFIGFILE $PROFILE $TESTS $SKIPS $SEVERITY $CONFIDENCE -x $INPUT_EXCLUDE $BASELINE $INI -r $INPUT_TARGETS -f sarif -o results.sarif || true`

All INPUT_* env vars are sourced from `inputs.*` (e.g. `INPUT_TARGETS: ${{ inputs.targets }}`), which are fully attacker-controlled. An attacker can inject shell metacharacters (`;`, `|`, `$(...)`, etc.) to execute arbitrary commands. Each expansion must be double-quoted, e.g. `"-c $INPUT_CONFIGFILE"` → `"-c \"$INPUT_CONFIGFILE\""` or restructured to quote all expansions.

Locations:

- `action.yml:84`
- `action.yml:88`
- `action.yml:92`
- `action.yml:96`
- `action.yml:100`
- `action.yml:104`
- `action.yml:108`
- `action.yml:112`
- `action.yml:113`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all three unpinned `uses:` references by replacing mutable tags with full 40-character SHA digests (actions/setup-python@v5→a26af69..., actions/checkout@v4→11d5960..., github/codeql-action/upload-sarif@v3→ce5e7ee...). Fixed script injection in the 'Run Bandit' step by replacing unquoted string variable assembly with a bash array (BANDIT_ARGS). Each optional flag and its value are now added as separate quoted array elements, and the final bandit invocation expands the array safely with "${BANDIT_ARGS[@]}" while also double-quoting $INPUT_EXCLUDE and $INPUT_TARGETS.

### Iteration 1

**Fixes applied:** publish-gate

**Notes:**

Fixed action.yml line 113: changed `"$INPUT_EXCLUDE"` and `"$INPUT_TARGETS"` to unquoted `$INPUT_EXCLUDE` and `$INPUT_TARGETS` in the bandit command. This allows the shell to word-split these values on whitespace, so multiple paths/patterns are passed as separate arguments to bandit, matching upstream behavior. Expanding env variables unquoted is not injection (it only word-splits and globs, nothing is evaluated).


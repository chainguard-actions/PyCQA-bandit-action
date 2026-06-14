<!-- markdownlint-disable -->

# Hardening Report: PyCQA--bandit-action/v1.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **PyCQA--bandit-action/v1.0.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Three `uses:` references in action.yml are pinned to mutable tags rather than immutable 40-character SHA digests, making the action vulnerable to supply-chain attacks if those tags are moved:
- `actions/setup-python@v5`
- `actions/checkout@v4`
- `github/codeql-action/upload-sarif@v3`
Each should be replaced with the corresponding full commit SHA.

Locations:

- `action.yml:62`
- `action.yml:68`
- `action.yml:113`

### script-injection (severity: high)

Rule (b) violation: The 'Run Bandit' step maps all user-controlled inputs into env vars (e.g., INPUT_CONFIGFILE, INPUT_PROFILE, INPUT_TARGETS, INPUT_EXCLUDE, etc.) but then expands those env vars **unquoted** throughout the run script. Unquoted expansions allow shell metacharacters (`;`, `|`, `&`, `$(...)`, whitespace, glob chars) embedded in attacker-supplied input values to be interpreted by the shell.

Specific failing lines:
- `CONFIGFILE="-c $INPUT_CONFIGFILE"` — unquoted expansion inside assignment
- `PROFILE="-p $INPUT_PROFILE"` — unquoted
- `TESTS="-t $INPUT_TESTS"` — unquoted
- `SKIPS="-s $INPUT_SKIPS"` — unquoted
- `SEVERITY="--severity-level $INPUT_SEVERITY"` — unquoted
- `CONFIDENCE="--confidence-level $INPUT_CONFIDENCE"` — unquoted
- `BASELINE="-b $INPUT_BASELINE"` — unquoted
- `INI="--ini $INPUT_INI"` — unquoted
- Final invocation: `bandit $CONFIGFILE $PROFILE $TESTS $SKIPS $SEVERITY $CONFIDENCE -x $INPUT_EXCLUDE $BASELINE $INI -r $INPUT_TARGETS` — all variables unquoted

All expansions should be double-quoted (e.g., `"$INPUT_CONFIGFILE"`, `"$INPUT_TARGETS"`).

Locations:

- `action.yml:72`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all three unpinned `uses:` references by replacing mutable tags with full 40-character SHA digests (setup-python@v5→a26af69, checkout@v4→34e1148, upload-sarif@v3→dd903d2). Fixed script injection by replacing the unquoted variable expansion pattern with a bash array approach: optional arguments are accumulated into BANDIT_ARGS with each value double-quoted, and the final bandit invocation uses "${BANDIT_ARGS[@]}" plus double-quoted "$INPUT_EXCLUDE" and "$INPUT_TARGETS", preventing shell metacharacter injection from user-controlled inputs.


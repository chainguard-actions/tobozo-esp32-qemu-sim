<!-- markdownlint-disable -->

# Hardening Report: tobozo--esp32-qemu-sim/v2.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tobozo--esp32-qemu-sim/v2.1.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All `uses:` references in action.yml use mutable tag refs instead of pinned 40-character SHA digests, making the action vulnerable to supply-chain attacks if the referenced tags are moved or compromised. Failing references: `actions/cache@v3` (×2), `actions/checkout@v3` (×2), `actions/setup-python@v5.4.0`, `actions/upload-artifact@v4`.

Locations:

- `action.yml:87`
- `action.yml:93`
- `action.yml:101`
- `action.yml:107`
- `action.yml:133`
- `action.yml:155`

### script-injection (severity: high)

Sub-rule (a): The `run:` field in the 'Run Build in QEmu' step directly interpolates a `${{ }}` expression: `run: ${{ github.action_path }}/run-build-in-qemu.sh`. Any `${{ ... }}` expression inside a `run:` block is subject to YAML template substitution before the shell processes it, making this a script-injection risk.

Locations:

- `action.yml:143`

### script-injection (severity: high)

Sub-rule (b): In `run-build-in-qemu.sh`, numerous `$ENV_*` variables — which are populated from `inputs.*` values via the `env:` block in action.yml — are expanded unquoted inside shell commands. Unquoted expansions allow shell metacharacters (`;`, `|`, `&`, `$(...)`, whitespace, globs) in attacker-controlled input values to be interpreted by the shell. Key offending lines include: `$ESPTOOL --chip $ENV_CHIP merge-bin --fill-flash-size ${ENV_FLASH_SIZE}MB -o $QEMU_FLASH_IMAGE ...` (all path and chip arguments unquoted); `$QEMU_BIN -nographic -machine $ENV_CHIP $ENV_PSRAM -drive file=$QEMU_FLASH_IMAGE,...` (chip, psram, and file path unquoted); `grep "${ENV_TIMEOUT_INT_RE}"` used in a backtick subshell and then `[[ "$grep_result" =~ $ENV_TIMEOUT_INT_RE ]]` with an unquoted regex pattern from user input.

Locations:

- `run-build-in-qemu.sh:109`
- `run-build-in-qemu.sh:133`
- `run-build-in-qemu.sh:155`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all 6 unpinned action references in action.yml with full SHA digests. Fixed script injection in action.yml by moving github.action_path into env block as ACTION_PATH and referencing it as $ACTION_PATH in the run field. Fixed script injection in run-build-in-qemu.sh by: (1) converting ENV_PSRAM from a string '-m SIZE' to a proper bash array PSRAM_ARGS=(-m "$ENV_PSRAM") to safely handle the multi-token value; (2) quoting all variables in the esptool command; (3) quoting all variables in the QEMU run command using "${PSRAM_ARGS[@]}" for the array; (4) using $() instead of backticks for grep_result and killall, with -E flag for grep and quoted log_file; (5) keeping =~ RHS unquoted as correct bash idiom for regex matching.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two unquoted variable expansions in hardened/action/run-build-in-qemu.sh: (1) Line 107: replaced backtick command substitution `cat $ENV_BUILD_FOLDER/$ENV_PARTITIONS_CSV` with $() syntax and double-quoted the path: `$(cat "$ENV_BUILD_FOLDER/$ENV_PARTITIONS_CSV" | ...)`. (2) Line 163: added double quotes to `sleep "$ENV_QEMU_TIMEOUT"`. Both variables are set from user-controlled inputs via the env: block in action.yml, so proper quoting prevents word-splitting and glob expansion attacks.


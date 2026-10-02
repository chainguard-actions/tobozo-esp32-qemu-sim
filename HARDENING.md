<!-- markdownlint-disable -->

# Hardening Report: tobozo--esp32-qemu-sim/v1.0.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tobozo--esp32-qemu-sim/v1.0.5** was hardened automatically. 2 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml references 6 GitHub Actions using mutable tags/version strings instead of immutable 40-character commit SHAs. If any of these actions are compromised or their tags are moved, the action will silently execute attacker-controlled code. Failing references: `actions/cache@v3` (used twice), `actions/checkout@v3` (used twice), `actions/setup-python@v5.4.0`, `actions/upload-artifact@v4`. Each should be pinned to a full SHA, e.g. `actions/cache@1bd1e32a3bdc45362d1e726936510720a7c6158d # v3`.

Locations:

- `action.yml:71`
- `action.yml:79`
- `action.yml:96`
- `action.yml:101`
- `action.yml:113`
- `action.yml:137`

### script-injection (severity: high)

Rule (a) violation: a `${{ ... }}` expression is interpolated directly inside a `run:` shell command string. The offending line is `run: ${{ github.action_path }}/run-build-in-qemu.sh`. Even though `github.action_path` is not attacker-controlled in the same way as `github.head_ref`, any `${{ ... }}` expression inside a `run:` block undergoes YAML template substitution before the shell sees it, making it a script-injection risk. The safe alternative is to use the `$GITHUB_ACTION_PATH` environment variable instead: `run: "$GITHUB_ACTION_PATH/run-build-in-qemu.sh"`.

Locations:

- `action.yml:133`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all 6 unpinned action references by pinning them to their full 40-character commit SHAs (actions/cache@v3 ×2, actions/checkout@v3 ×2, actions/setup-python@v5.4.0, actions/upload-artifact@v4). Fixed script injection by replacing `${{ github.action_path }}/run-build-in-qemu.sh` with `"$GITHUB_ACTION_PATH/run-build-in-qemu.sh"` using the built-in environment variable.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed all four script-injection locations in run-build-in-qemu.sh:
1. csvdata assignment (line 61): Replaced backtick substitution with $() and quoted path variables: `$(cat "$ENV_BUILD_FOLDER/$ENV_PARTITIONS_CSV" | ...)`
2. esptool command (line 107): Double-quoted all ENV_* variables: "$ENV_BOOTLOADER_ADDR", "$ENV_BUILD_FOLDER/$ENV_BOOTLOADER_BIN", "$ENV_PARTITIONS_ADDR", "$ENV_BUILD_FOLDER/$ENV_PARTITIONS_BIN", "$OTADATA_ADDR", "$ENV_BUILD_FOLDER/$ENV_OTADATA_BIN", "$FIRMWARE_ADDR", "$ENV_BUILD_FOLDER/$ENV_FIRMWARE_BIN", "$SPIFFS_ADDR", "$ENV_BUILD_FOLDER/$ENV_SPIFFS_BIN", and "${ENV_FLASH_SIZE}MB"
3. QEMU command (line 119): Replaced unquoted $ENV_PSRAM (which held either empty string or "-m SIZE" two-token string) with a bash array PSRAM_ARGS built safely at validation time, expanded as "${PSRAM_ARGS[@]}"
4. grep_result assignment (line 131): Replaced backtick substitution with $() and quoted "$log_file"

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed three unquoted shell variable expansions in hardened/action/run-build-in-qemu.sh:
1. Line 101: Changed backtick command substitution `cat $ENV_BUILD_FOLDER/$ENV_PARTITIONS_CSV` to $(cat "$ENV_BUILD_FOLDER/$ENV_PARTITIONS_CSV") — properly quotes both $ENV_BUILD_FOLDER and $ENV_PARTITIONS_CSV and uses modern $() syntax.
2. Line 132: Changed `=~ $ENV_TIMEOUT_INT_RE` to `=~ "$ENV_TIMEOUT_INT_RE"` — quotes the regex variable in the [[ ]] match context to prevent metacharacter injection.
3. Line 143: Changed `sleep $ENV_QEMU_TIMEOUT` to `sleep "$ENV_QEMU_TIMEOUT"` — quotes the variable to prevent word-splitting.


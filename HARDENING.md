<!-- markdownlint-disable -->

# Hardening Report: tobozo--esp32-qemu-sim/v2.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **tobozo--esp32-qemu-sim/v2.0.0** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All 6 `uses:` references in action.yml use mutable tags instead of pinned 40-character SHA commit hashes, making the action vulnerable to supply-chain attacks if any upstream action is compromised or its tag is moved. Failing references: `actions/cache@v3` (×2), `actions/checkout@v3` (×2), `actions/setup-python@v5.4.0`, `actions/upload-artifact@v4`.

Locations:

- `action.yml:68`
- `action.yml:77`
- `action.yml:107`
- `action.yml:113`
- `action.yml:127`
- `action.yml:163`

### script-injection (severity: high)

Sub-rule (a): A `${{ }}` expression is interpolated directly inside a `run:` shell command string. The line `run: ${{ github.action_path }}/run-build-in-qemu.sh` embeds `${{ github.action_path }}` directly in the shell command. Any `${{ ... }}` expression inside a `run:` block is a script-injection risk because the value is substituted by the template engine before the shell ever sees it, bypassing shell quoting.

Locations:

- `action.yml:155`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all 6 unpinned `uses:` references by resolving each tag to its full 40-character SHA (actions/cache@v3→6f8efc29, actions/checkout@v3→f43a0e5f, actions/setup-python@v5.4.0→42375524, actions/upload-artifact@v4→ea165f8d). Fixed the script-injection finding by moving `${{ github.action_path }}` from the `run:` shell string into the step's `env:` block as `ACTION_PATH`, then referencing it as `$ACTION_PATH` in the shell command. The new env var was merged into the existing `env:` block to avoid duplicate YAML keys.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed all unquoted ENV_* variable expansions in run-build-in-qemu.sh: (1) Line 96: Quoted $ENV_BUILD_FOLDER and $ENV_PARTITIONS_CSV in the cat command. (2) Line 131: Quoted all ENV_* variables in the esptool merge-bin command including $ENV_CHIP, ${ENV_FLASH_SIZE}MB, path variables, and address variables. (3) Line 143: Quoted $QEMU_BIN, $ENV_CHIP, and $log_file in the qemu command; replaced unquoted $ENV_PSRAM with a bash array PSRAM_ARGS=("-m" "$ENV_PSRAM") and used "${PSRAM_ARGS[@]}" to safely handle the two-word expansion. (4) Line 155: Quoted ${log_file} in the tail command and $ENV_TIMEOUT_INT_RE in the grep command. (5) Additional fixes: quoted $ENV_BUILD_FOLDER/$ENV_PARTITIONS_CSV in the debug cat call, quoted $QEMU_BIN in the killall basename call, and quoted $log_file in the tee -a call.


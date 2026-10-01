<!-- markdownlint-disable -->

# Hardening Report: tobozo--esp32-qemu-sim/v2.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tobozo--esp32-qemu-sim/v2.0.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All 6 `uses:` references in action.yml are pinned to mutable tags rather than immutable 40-character SHA digests. This exposes the action to supply-chain attacks if any upstream action is compromised or its tag is moved. Failing references: `actions/cache@v3` (×2), `actions/checkout@v3` (×2), `actions/setup-python@v5.4.0`, `actions/upload-artifact@v4`.

Locations:

- `action.yml:68`
- `action.yml:77`
- `action.yml:91`
- `action.yml:99`
- `action.yml:108`
- `action.yml:130`

### script-injection (severity: high)

Sub-rule (a): The `run:` value in the 'Run Build in QEmu' step directly interpolates a `${{ }}` expression: `run: ${{ github.action_path }}/run-build-in-qemu.sh`. Any `${{ ... }}` expression inside a `run:` string is substituted by the Actions template engine before the shell ever sees it, making it a script-injection vector. Even though `github.action_path` is not attacker-controlled in isolation, this pattern is flagged because the rule prohibits any `${{ }}` inside a `run:` block.

Locations:

- `action.yml:122`

### script-injection (severity: high)

Sub-rule (b): In `run-build-in-qemu.sh`, numerous `ENV_*` variables that are populated directly from `inputs.*` (via the `env:` block in action.yml) are expanded unquoted in shell commands. Unquoted expansions allow shell metacharacter injection (`;`, `|`, `&`, `$(...)`, glob chars, whitespace splitting) from attacker-supplied input values. Key offending lines include:
- `$ESPTOOL --chip $ENV_CHIP merge-bin --fill-flash-size ${ENV_FLASH_SIZE}MB ...` (unquoted $ENV_CHIP, $ENV_FLASH_SIZE, $ENV_BUILD_FOLDER, $ENV_BOOTLOADER_BIN, $ENV_PARTITIONS_ADDR, $ENV_PARTITIONS_BIN, $ENV_OTADATA_BIN, $ENV_FIRMWARE_BIN, $ENV_SPIFFS_BIN)
- `$QEMU_BIN -nographic -machine $ENV_CHIP $ENV_PSRAM -drive file=flash_image.bin,...` (unquoted $ENV_CHIP, $ENV_PSRAM)
- `sleep $ENV_QEMU_TIMEOUT` (unquoted)
- `grep "${ENV_TIMEOUT_INT_RE}"` used unquoted in `=~` pattern test: `[[ "$grep_result" =~ $ENV_TIMEOUT_INT_RE ]]`
All these variables must be double-quoted: `"$ENV_CHIP"`, `"$ENV_PSRAM"`, etc.

Locations:

- `run-build-in-qemu.sh:118`
- `run-build-in-qemu.sh:119`
- `run-build-in-qemu.sh:120`
- `run-build-in-qemu.sh:121`
- `run-build-in-qemu.sh:122`
- `run-build-in-qemu.sh:136`
- `run-build-in-qemu.sh:137`
- `run-build-in-qemu.sh:148`
- `run-build-in-qemu.sh:155`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all 6 unpinned `uses:` references in action.yml by pinning to full 40-char SHA digests (actions/cache@6f8efc29..., actions/checkout@a37ce912..., actions/setup-python@42375524..., actions/upload-artifact@ea165f8d...). Fixed script-injection in action.yml by moving `${{ github.action_path }}` out of the `run:` block into the `env:` block as `ACTION_PATH`, then referencing it as `"$ACTION_PATH/run-build-in-qemu.sh"`. Fixed script-injection in run-build-in-qemu.sh by quoting all unquoted ENV_* variables in the esptool command, handling the two-token `$ENV_PSRAM` via a bash array, quoting `"$ENV_CHIP"` in the QEMU command, quoting `sleep "$ENV_QEMU_TIMEOUT"` and `sleep "$interval"`, and replacing backtick command substitution with `$()` with proper quoting. The `=~ $ENV_TIMEOUT_INT_RE` RHS in the bash `[[ ]]` test is intentionally left unquoted as quoting it would treat it as a literal string rather than a regex pattern.


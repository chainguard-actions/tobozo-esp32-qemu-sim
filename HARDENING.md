<!-- markdownlint-disable -->

# Hardening Report: tobozo--esp32-qemu-sim/v2.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **tobozo--esp32-qemu-sim/v2.1.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All 6 `uses:` references in action.yml use mutable version tags instead of pinned full-length SHA digests, making the action vulnerable to supply-chain attacks if any of those upstream actions are compromised or their tags are moved. Failing references:
- `actions/cache@v3` (Cache QEmu build step)
- `actions/checkout@v3` (Checkout qemu-xtensa step)
- `actions/setup-python@v5.4.0` (Setup Python 3.11 step)
- `actions/cache@v3` (Cache pip step)
- `actions/checkout@v3` (Checkout esptool.py step)
- `actions/upload-artifact@v4` (Upload logs as artifact step)

Locations:

- `action.yml:80`
- `action.yml:87`
- `action.yml:100`
- `action.yml:107`
- `action.yml:116`
- `action.yml:136`

### script-injection (severity: high)

Sub-rule (a): The `run:` field of the 'Run Build in QEmu' step directly interpolates a `${{ }}` expression: `run: ${{ github.action_path }}/run-build-in-qemu.sh`. Any `${{ ... }}` expression inside a `run:` block is a script-injection risk because the value is substituted by the YAML template engine before the shell ever sees it. The safe alternative is to use the `$GITHUB_ACTION_PATH` environment variable instead.

Locations:

- `action.yml:134`

### script-injection (severity: high)

Sub-rule (b): Multiple ENV_* shell variables — all sourced from `inputs.*` via the `env:` block in action.yml — are expanded unquoted inside `run-build-in-qemu.sh`, allowing shell metacharacter injection. Specific violations:
1. Line ~143: `$ESPTOOL --chip $ENV_CHIP merge-bin --fill-flash-size ${ENV_FLASH_SIZE}MB -o $QEMU_FLASH_IMAGE $ENV_BOOTLOADER_ADDR $ENV_BUILD_FOLDER/$ENV_BOOTLOADER_BIN ...` — $ENV_CHIP, $ENV_FLASH_SIZE, $ENV_BUILD_FOLDER, $ENV_BOOTLOADER_BIN, $ENV_PARTITIONS_ADDR, $ENV_PARTITIONS_BIN, $ENV_FIRMWARE_BIN, $ENV_SPIFFS_BIN are all unquoted.
2. Line ~155: `($QEMU_BIN -nographic -machine $ENV_CHIP $ENV_PSRAM -drive file=$QEMU_FLASH_IMAGE,...)` — $ENV_CHIP and $ENV_PSRAM are unquoted.
3. Line ~108: `csvdata=\`cat $ENV_BUILD_FOLDER/$ENV_PARTITIONS_CSV | tr -d ' ' | tr '\n' ';'\`` — $ENV_BUILD_FOLDER and $ENV_PARTITIONS_CSV are unquoted in a backtick command substitution.
4. Line ~175: `grep_result=\`tail ${log_file} | grep "${ENV_TIMEOUT_INT_RE}"\`` and `[[ "$grep_result" =~ $ENV_TIMEOUT_INT_RE ]]` — $ENV_TIMEOUT_INT_RE is unquoted in the =~ operator, allowing regex injection.

Locations:

- `run-build-in-qemu.sh:143`
- `run-build-in-qemu.sh:155`
- `run-build-in-qemu.sh:108`
- `run-build-in-qemu.sh:175`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all 6 unpinned `uses:` references in action.yml by pinning to full commit SHAs: actions/cache@v3→6f8efc29b200d32929f49075959781ed54ec270c (×2), actions/checkout@v3→f43a0e5ff2bd294095638e18286ca9a3d1956744 (×2), actions/setup-python@v5.4.0→42375524e23c412d93fb67b49958b491fce71c38, actions/upload-artifact@v4→ea165f8d65b6e75b540449e92b4886f43607fa02. Fixed script injection in action.yml by replacing `${{ github.action_path }}` with `$GITHUB_ACTION_PATH`. Fixed script injection in run-build-in-qemu.sh: (1) quoted $ENV_BUILD_FOLDER/$ENV_PARTITIONS_CSV in csvdata command substitution and modernized to $(); (2) quoted all ENV_* variables in the esptool merge-bin command; (3) replaced unquoted $ENV_CHIP/$ENV_PSRAM in QEMU command with properly quoted variables and an array for PSRAM args to handle word-splitting safely; (4) replaced backtick grep with $() and grep -F for literal matching, stored $ENV_TIMEOUT_INT_RE in a local variable before use in =~ operator; (5) fixed killall backtick to use $() with proper quoting.


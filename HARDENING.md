<!-- markdownlint-disable -->

# Hardening Report: tobozo--esp32-qemu-sim/v1.0.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tobozo--esp32-qemu-sim/v1.0.4** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml references multiple actions using mutable tags/versions instead of pinned full-length SHA commit hashes. This exposes the action to supply-chain attacks if the upstream tag is moved or overwritten. Failing references: actions/cache@v3 (line 68), actions/checkout@v3 (line 78), actions/setup-python@v5.3.0 (line 96), actions/cache@v3 (line 101), actions/checkout@v3 (line 112), actions/upload-artifact@v4 (line 148).

Locations:

- `action.yml:68`
- `action.yml:78`
- `action.yml:96`
- `action.yml:101`
- `action.yml:112`
- `action.yml:148`

### script-injection (severity: high)

Rule (a): The run: field directly interpolates a GitHub Actions expression inside a shell command string: `run: ${{ github.action_path }}/run-build-in-qemu.sh`. Any ${{ ... }} expression inside a run: block is a script-injection risk because the value is substituted by the template engine before the shell ever sees it, allowing an attacker-controlled value to alter the command being executed.

Locations:

- `action.yml:133`

### script-injection (severity: high)

Rule (b): The shell script run-build-in-qemu.sh uses numerous unquoted expansions of $ENV_* variables (e.g. $ENV_BUILD_FOLDER, $ENV_FLASH_SIZE, $ENV_PSRAM, $ENV_QEMU_TIMEOUT, $ENV_BOOTLOADER_ADDR, $ENV_BOOTLOADER_BIN, $ENV_PARTITIONS_ADDR, $ENV_PARTITIONS_CSV, $ENV_SPIFFS_ADDR, $ENV_SPIFFS_BIN, $ENV_OTADATA_BIN, $ENV_FIRMWARE_BIN, $ENV_PARTITIONS_BIN) that are populated directly from inputs.* values via the env: block in action.yml. Unquoted shell expansions allow shell metacharacters (;, |, &, $(...), whitespace, glob chars) in the input values to be interpreted by the shell, enabling command injection. Examples of failing lines include: `$ESPTOOL_PY --chip esp32 merge_bin --fill-flash-size ${ENV_FLASH_SIZE}MB ...` and `($QEMU_BIN -nographic -machine esp32 $ENV_PSRAM ...)` and path constructions like `$ENV_BUILD_FOLDER/$ENV_PARTITIONS_CSV`.

Locations:

- `run-build-in-qemu.sh:16`
- `run-build-in-qemu.sh:30`
- `run-build-in-qemu.sh:31`
- `run-build-in-qemu.sh:32`
- `run-build-in-qemu.sh:36`
- `run-build-in-qemu.sh:40`
- `run-build-in-qemu.sh:44`
- `run-build-in-qemu.sh:47`
- `run-build-in-qemu.sh:48`
- `run-build-in-qemu.sh:49`
- `run-build-in-qemu.sh:50`
- `run-build-in-qemu.sh:95`
- `run-build-in-qemu.sh:96`
- `run-build-in-qemu.sh:97`
- `run-build-in-qemu.sh:98`
- `run-build-in-qemu.sh:99`
- `run-build-in-qemu.sh:103`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all 6 unpinned action references in action.yml by resolving full SHA hashes: actions/cache@v3→6f8efc29, actions/checkout@v3→a37ce912, actions/setup-python@v5.3.0→0b93645e, actions/upload-artifact@v4→ea165f8d. Fixed script-injection in action.yml line 133 by moving ${{ github.action_path }} into the env: block as ACTION_PATH and referencing it as "$ACTION_PATH/run-build-in-qemu.sh" in run:. Fixed script-injection in run-build-in-qemu.sh by double-quoting all $ENV_* variable expansions throughout the script; converted $ENV_PSRAM handling to a bash array PSRAM_FLAGS=("-m" "$ENV_PSRAM") to safely handle the two-token flag case; quoted all path constructions, esptool/qemu command arguments, and sleep argument.


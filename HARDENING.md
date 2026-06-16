<!-- markdownlint-disable -->

# Hardening Report: tobozo--esp32-qemu-sim/v1.0.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **tobozo--esp32-qemu-sim/v1.0.4** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A ${{ }} expression is interpolated directly inside a `run:` shell command string. The line `run: ${{ github.action_path }}/run-build-in-qemu.sh` embeds the expression directly in the run value, which is evaluated by the YAML template engine before the shell sees it. Any `${{ ... }}` directly inside a `run:` block is a script-injection finding per the check rules.

Locations:

- `action.yml:118`

### unpinned-uses (severity: high)

All 6 `uses:` references in action.yml use mutable tags or version strings instead of full 40-character commit SHA digests, making the action vulnerable to supply-chain attacks if any upstream action is compromised or its tag is moved. Failing references: `actions/cache@v3` (lines 57, 88), `actions/checkout@v3` (lines 68, 98), `actions/setup-python@v5.3.0` (line 83), `actions/upload-artifact@v4` (line 122).

Locations:

- `action.yml:57`
- `action.yml:68`
- `action.yml:83`
- `action.yml:88`
- `action.yml:98`
- `action.yml:122`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses

**Notes:**

Fixed script-injection by moving `${{ github.action_path }}` from the `run:` shell string into the step's `env:` block as `ACTION_PATH`, then referencing it as `"$ACTION_PATH/run-build-in-qemu.sh"` in the run command. Fixed all 6 unpinned `uses:` references by pinning to full 40-character commit SHAs: actions/cache@v3 (×2) → 6f8efc29b200d32929f49075959781ed54ec270c, actions/checkout@v3 (×2) → f43a0e5ff2bd294095638e18286ca9a3d1956744, actions/setup-python@v5.3.0 → 0b93645e9fea7318ecaed2b359559ac225c90a2b, actions/upload-artifact@v4 → ea165f8d65b6e75b540449e92b4886f43607fa02. All original tag names preserved as comments.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed all unquoted variable expansions in run-build-in-qemu.sh:

1. **Line ~72 (csvdata)**: Changed backtick command substitution to `$(...)` and quoted the path: `cat "$ENV_BUILD_FOLDER/$ENV_PARTITIONS_CSV"`.

2. **Line ~116 (esptool command)**: Quoted `"$ESPTOOL_PY"`, `"${ENV_FLASH_SIZE}MB"`, `"$ENV_BOOTLOADER_ADDR"`, `"$ENV_BUILD_FOLDER/$ENV_BOOTLOADER_BIN"`, `"$ENV_PARTITIONS_ADDR"`, `"$ENV_BUILD_FOLDER/$ENV_PARTITIONS_BIN"`, `"$OTADATA_ADDR"`, `"$ENV_BUILD_FOLDER/$ENV_OTADATA_BIN"`, `"$FIRMWARE_ADDR"`, `"$ENV_BUILD_FOLDER/$ENV_FIRMWARE_BIN"`, `"$SPIFFS_ADDR"`, `"$ENV_BUILD_FOLDER/$ENV_SPIFFS_BIN"`.

3. **Line ~124 (QEMU command)**: `$ENV_PSRAM` was previously expanded unquoted as a string like `-m 2M` (two words). Replaced with a bash array `PSRAM_ARGS`: when PSRAM matches `^(2M|4M)$`, set `PSRAM_ARGS=("-m" "$ENV_PSRAM")`, otherwise `PSRAM_ARGS=()`. Used `"${PSRAM_ARGS[@]}"` in the QEMU invocation and quoted `"$QEMU_BIN"`.

4. **Line ~126 (sleep)**: Quoted `sleep "$ENV_QEMU_TIMEOUT"`.

5. Also fixed `_debug` calls that used backtick substitution with unquoted paths: converted to `$(...)` with proper quoting.

All inputs are validated with strict regex patterns before use (flash size, timeout, addresses), providing defense-in-depth. The `$ENV_PSRAM` handling via array is the correct approach since it may expand to zero or two words.


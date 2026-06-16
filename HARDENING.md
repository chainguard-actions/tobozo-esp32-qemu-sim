<!-- markdownlint-disable -->

# Hardening Report: tobozo--esp32-qemu-sim/v1.0.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **tobozo--esp32-qemu-sim/v1.0.5** was hardened automatically. 2 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All six `uses:` references in action.yml use mutable version tags instead of full 40-character SHA commit digests. This exposes the action to supply-chain attacks where a tag could be silently moved to point to malicious code. Failing references: `actions/cache@v3` (×2), `actions/checkout@v3` (×2), `actions/setup-python@v5.4.0`, `actions/upload-artifact@v4`. Each should be pinned to a full SHA, e.g. `actions/cache@1bd1e32a3bdc45362d1e726936510720a7c6158d # v3`.

Locations:

- `action.yml:68`
- `action.yml:75`
- `action.yml:91`
- `action.yml:97`
- `action.yml:107`
- `action.yml:148`

### script-injection (severity: high)

Sub-rule (a): A `${{ }}` expression is interpolated directly inside a `run:` shell command string. The offending line is: `run: ${{ github.action_path }}/run-build-in-qemu.sh`. Any `${{ ... }}` expression inside a `run:` block undergoes YAML template substitution before the shell sees it, bypassing shell quoting. The safe alternative is to use the pre-set environment variable `$GITHUB_ACTION_PATH` instead: `run: "$GITHUB_ACTION_PATH/run-build-in-qemu.sh"`.

Locations:

- `action.yml:146`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all 6 unpinned `uses:` references by pinning them to full 40-character SHA digests (actions/cache@v3 ×2 → SHA 6f8efc29..., actions/checkout@v3 ×2 → SHA f43a0e5f..., actions/setup-python@v5.4.0 → SHA 42375524..., actions/upload-artifact@v4 → SHA ea165f8d...). Fixed script-injection by replacing `${{ github.action_path }}/run-build-in-qemu.sh` in the `run:` block with `"$GITHUB_ACTION_PATH/run-build-in-qemu.sh"`, using the pre-set environment variable instead of a template expression.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted variable expansions in run-build-in-qemu.sh at the two identified command execution sites:
1. Replaced the unquoted `$ESPTOOL` invocation (line ~100) with a bash array `esptool_cmd` where all ENV_* variables (ENV_BOOTLOADER_ADDR, ENV_BUILD_FOLDER, ENV_BOOTLOADER_BIN, ENV_PARTITIONS_ADDR, ENV_PARTITIONS_BIN, ENV_FIRMWARE_BIN, ENV_SPIFFS_BIN, ENV_FLASH_SIZE) are properly double-quoted.
2. Replaced the unquoted `$QEMU_BIN` invocation (line ~108) with a bash array `qemu_cmd` where ENV_PSRAM is safely split into a `qemu_psram_args` array via `read -ra` (safe because ENV_PSRAM is already sanitized to either empty or '-m 2M'/'-m 4M' by the regex check earlier in the script), then expanded as "${qemu_psram_args[@]}".
The action.yml file did not require changes as it correctly passes inputs via the env: block rather than directly in the run: command.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed script injection vulnerability in run-build-in-qemu.sh line 62: replaced unquoted backtick command substitution `cat $ENV_BUILD_FOLDER/$ENV_PARTITIONS_CSV` with properly quoted $() form: $(cat "$ENV_BUILD_FOLDER/$ENV_PARTITIONS_CSV" | tr -d ' ' | tr '\n' ';'). The double quotes around the path variable prevent shell metacharacters (;, |, &, $(...), whitespace) in ENV_BUILD_FOLDER or ENV_PARTITIONS_CSV from being interpreted as shell commands.


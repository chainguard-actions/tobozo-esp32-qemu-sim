<!-- markdownlint-disable -->

# Hardening Report: tobozo--esp32-qemu-sim/v2.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tobozo--esp32-qemu-sim/v2.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml references five actions using mutable tag/version refs instead of pinned 40-character SHA commit hashes. This exposes the workflow to supply-chain attacks if any of those tags are moved or compromised. Failing references: actions/cache@v3 (×2), actions/checkout@v3 (×2), actions/setup-python@v5.4.0, actions/upload-artifact@v4.

Locations:

- `action.yml:72`
- `action.yml:82`
- `action.yml:103`
- `action.yml:110`
- `action.yml:122`
- `action.yml:148`

### script-injection (severity: high)

Rule (a): The `run:` block in the 'Run Build in QEmu' step directly interpolates the expression `${{ github.action_path }}` into the shell command string: `run: ${{ github.action_path }}/run-build-in-qemu.sh`. Any `${{ ... }}` expression inside a `run:` block is subject to YAML template substitution before the shell sees it, making this a script-injection risk.

Locations:

- `action.yml:143`

### script-injection (severity: high)

Rule (b): run-build-in-qemu.sh uses numerous unquoted `$ENV_*` shell variable expansions in command invocations. These variables are populated directly from `inputs.*` values via the `env:` block in action.yml and are therefore workflow-controllable. Unquoted expansions allow shell metacharacters (`;`, `|`, `&`, `$(...)`, whitespace, globs) embedded in input values to be interpreted by the shell. Examples of unquoted uses: `$ENV_CHIP` in `case $ENV_CHIP`, in the qemu invocation `-machine $ENV_CHIP`, and in `$ESPTOOL --chip $ENV_CHIP merge-bin ...`; `$ENV_PSRAM` in `-machine $ENV_CHIP $ENV_PSRAM`; `$ENV_BUILD_FOLDER`, `$ENV_BOOTLOADER_BIN`, `$ENV_PARTITIONS_BIN`, `$ENV_OTADATA_BIN`, `$ENV_FIRMWARE_BIN`, `$ENV_SPIFFS_BIN` all passed unquoted to esptool; `$ENV_QEMU_TIMEOUT` passed unquoted to `sleep`; `${ENV_TIMEOUT_INT_RE}` used unquoted in `grep`. All of these should be double-quoted.

Locations:

- `run-build-in-qemu.sh:30`
- `run-build-in-qemu.sh:119`
- `run-build-in-qemu.sh:121`
- `run-build-in-qemu.sh:122`
- `run-build-in-qemu.sh:123`
- `run-build-in-qemu.sh:124`
- `run-build-in-qemu.sh:125`
- `run-build-in-qemu.sh:131`
- `run-build-in-qemu.sh:143`
- `run-build-in-qemu.sh:148`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all 6 unpinned action references in action.yml by pinning to full commit SHAs (actions/cache@v3→6f8efc29, actions/checkout@v3→a37ce912, actions/setup-python@v5.4.0→42375524, actions/upload-artifact@v4→ea165f8d). Fixed script injection in action.yml by moving ${{ github.action_path }} to env block as ACTION_PATH and referencing it as "$ACTION_PATH/run-build-in-qemu.sh" in run:. Fixed script injection in run-build-in-qemu.sh by double-quoting all unquoted $ENV_* variables in command invocations (esptool call, QEMU invocation, sleep, log_file, csvdata). Refactored $ENV_PSRAM from a string with embedded spaces to a bash array (PSRAM_ARGS) to safely handle the optional -m flag in the QEMU invocation.


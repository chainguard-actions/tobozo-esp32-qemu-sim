<!-- markdownlint-disable -->

# Hardening Report: tobozo--esp32-qemu-sim/v1.0.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tobozo--esp32-qemu-sim/v1.0.5** was hardened automatically. 3 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable tags or version strings instead of immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks.

In action.yml:
- `actions/cache@v3`
- `actions/checkout@v3` (used twice)
- `actions/setup-python@v5.4.0`
- `actions/upload-artifact@v4`

In .github/workflows/test.yml:
- `actions/checkout@v3`
- `ArminJo/arduino-test-compile@v3.2.0`
- `tobozo/esp32-qemu-sim@main` (mutable branch reference)

Locations:

- `action.yml:63`
- `action.yml:72`
- `action.yml:89`
- `action.yml:97`
- `action.yml:107`
- `action.yml:155`
- `.github/workflows/test.yml:17`
- `.github/workflows/test.yml:21`
- `.github/workflows/test.yml:31`

### script-injection (severity: high)

Sub-rule (a): A `${{ }}` expression is interpolated directly inside a `run:` shell command string in action.yml. The line `run: ${{ github.action_path }}/run-build-in-qemu.sh` injects the `github.action_path` context value directly into the shell command before the shell ever sees it. Any `${{ ... }}` in a `run:` block is a script-injection risk regardless of which context it reads from.

Sub-rule (b): The shell script `run-build-in-qemu.sh` (invoked by the `run:` step) expands multiple `$ENV_*` variables — which are set from `inputs.*` values — without double-quoting them in shell commands. Examples of unquoted expansions:
- `$ESPTOOL --chip esp32 merge-bin --fill-flash-size ${ENV_FLASH_SIZE}MB -o flash_image.bin $ENV_BOOTLOADER_ADDR $ENV_BUILD_FOLDER/$ENV_BOOTLOADER_BIN ...` (line ~113)
- `$QEMU_BIN -nographic -machine esp32 $ENV_PSRAM -drive ...` (line ~125)
- `` grep_result=`tail ${log_file} | grep "${ENV_TIMEOUT_INT_RE}"` `` — `ENV_TIMEOUT_INT_RE` is used as a grep pattern without quoting in the backtick context and in the `=~` test without quotes (line ~138)
Unquoted shell variables holding user-controlled data allow shell metacharacter injection.

Locations:

- `action.yml:131`
- `run-build-in-qemu.sh:113`
- `run-build-in-qemu.sh:125`
- `run-build-in-qemu.sh:138`

### missing-permissions (severity: medium)

The workflow file `.github/workflows/test.yml` has no top-level `permissions:` key and no job-level `permissions:` key on any job. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad (e.g., `write` access to contents). A minimal `permissions:` block should be added.

Locations:

- `.github/workflows/test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings:

1. **unpinned-uses**: Pinned all `uses:` references to full 40-char SHAs:
   - action.yml: `actions/cache@v3` → `@6f8efc29b200d32929f49075959781ed54ec270c` (×2), `actions/checkout@v3` → `@a37ce9120846195fa4ece8f58b268e6043cb2f26` (×2), `actions/setup-python@v5.4.0` → `@42375524e23c412d93fb67b49958b491fce71c38`, `actions/upload-artifact@v4` → `@ea165f8d65b6e75b540449e92b4886f43607fa02`
   - test.yml: `actions/checkout@v3` → `@a37ce9120846195fa4ece8f58b268e6043cb2f26`, `ArminJo/arduino-test-compile@v3.2.0` → `@f62f0163b11302441f58b50f8905a495f01ce858`, `tobozo/esp32-qemu-sim@main` → `@c791ac713f8024b900812479e6d27305f5983db8`

2. **script-injection**: 
   - action.yml: Moved `${{ github.action_path }}` out of `run:` into `env: ACTION_PATH:`, then referenced as `$ACTION_PATH` in the shell command.
   - run-build-in-qemu.sh: Double-quoted all `$ENV_*` variables in the esptool command; used a bash array for QEMU args to safely handle optional `$ENV_PSRAM`; used `grep -F` (fixed-string) and `$()` syntax for the timeout grep.

3. **missing-permissions**: Added `permissions: {}` at the top level of `.github/workflows/test.yml`.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed three unquoted variable expansions in run-build-in-qemu.sh:
1. Line 57: Added double-quotes around `$ENV_BUILD_FOLDER/$ENV_PARTITIONS_CSV` inside the backtick command substitution (`cat "$ENV_BUILD_FOLDER/$ENV_PARTITIONS_CSV"`).
2. Line 97 (PSRAM handling): Replaced the single string variable `ENV_PSRAM` (which was set to '-m 2M' or '') with a proper bash array `psram_args=(-m "$ENV_PSRAM")` or `psram_args=()`. The array is safely appended to `qemu_args` via `"${psram_args[@]}"`, eliminating the unquoted array expansion.
3. Line 113: Added double-quotes around `$ENV_QEMU_TIMEOUT` in the `sleep "$ENV_QEMU_TIMEOUT"` call.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted shell variable expansions in run-build-in-qemu.sh at line 88. Replaced backtick command substitution `cat $ENV_BUILD_FOLDER/$ENV_PARTITIONS_CSV` with modern $() syntax and properly double-quoted the variables inside: $(cat "$ENV_BUILD_FOLDER/$ENV_PARTITIONS_CSV"). This prevents word splitting and glob expansion on attacker-controlled values from inputs.build-folder and inputs.partitions-csv.


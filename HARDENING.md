<!-- markdownlint-disable -->

# Hardening Report: tobozo--esp32-qemu-sim/v2.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tobozo--esp32-qemu-sim/v2.0.1** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A ${{ }} expression is interpolated directly inside a `run:` shell command string in action.yml. The line `run: ${{ github.action_path }}/run-build-in-qemu.sh` embeds `${{ github.action_path }}` directly in the run: value. Although `github.action_path` is not attacker-controlled in the same way as `github.head_ref`, any `${{ ... }}` expression inside a `run:` block is a script-injection finding per the check rules, as the value flows through YAML template substitution before the shell processes it. The safe alternative is to use the `$GITHUB_ACTION_PATH` environment variable instead.

Locations:

- `action.yml:107`

### unpinned-uses (severity: high)

Multiple `uses:` references in action.yml and workflow files are pinned to mutable tags or branch names instead of immutable 40-character SHA digests, making the action vulnerable to supply-chain attacks if those tags are moved or the branches are updated. Failing references in action.yml: `actions/cache@v3`, `actions/checkout@v3` (×2), `actions/setup-python@v5.4.0`, `actions/cache@v3`, `actions/upload-artifact@v4`. Failing references in workflow files: `actions/checkout@v3`, `ArminJo/arduino-test-compile@v3.2.0`, `tobozo/esp32-qemu-sim@main` (in all three workflow files).

Locations:

- `action.yml:63`
- `action.yml:70`
- `action.yml:84`
- `action.yml:90`
- `action.yml:100`
- `action.yml:117`
- `.github/workflows/test-esp32.yml:16`
- `.github/workflows/test-esp32.yml:22`
- `.github/workflows/test-esp32.yml:30`
- `.github/workflows/test-esp32c3.yml:16`
- `.github/workflows/test-esp32c3.yml:22`
- `.github/workflows/test-esp32c3.yml:30`
- `.github/workflows/test-esp32s3.yml:16`
- `.github/workflows/test-esp32s3.yml:22`
- `.github/workflows/test-esp32s3.yml:30`

### missing-permissions (severity: medium)

None of the three workflow files define a top-level `permissions:` key, and no job within them defines a `permissions:` key either. Without explicit permissions, workflows run with the default token permissions (which may be `write-all` depending on repository settings), granting unnecessarily broad access. Each workflow should declare minimal required permissions (e.g., `permissions: contents: read`).

Locations:

- `.github/workflows/test-esp32.yml:1`
- `.github/workflows/test-esp32c3.yml:1`
- `.github/workflows/test-esp32s3.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings: (1) script-injection in action.yml line 107 - replaced `${{ github.action_path }}` in run: with `$GITHUB_ACTION_PATH` environment variable; (2) unpinned-uses - pinned all 6 distinct action references to full SHA digests in action.yml and all 3 workflow files; (3) missing-permissions - added `permissions: contents: read` top-level block to test-esp32.yml, test-esp32c3.yml, and test-esp32s3.yml.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed all unquoted ENV_* variable expansions in run-build-in-qemu.sh:
1. Quoted $ENV_CHIP, ${ENV_FLASH_SIZE}MB, and all $ENV_BUILD_FOLDER/$ENV_*_BIN path arguments in the esptool merge-bin command (lines ~163-168).
2. Replaced the ENV_PSRAM string variable (which held '-m 2M' — two shell tokens) with a bash array PSRAM_ARGS=(-m "$ENV_PSRAM") expanded safely as "${PSRAM_ARGS[@]}" in the QEMU command (line ~176).
3. Quoted "$ENV_CHIP" in the QEMU -machine argument and "$log_file" in the tee command.
4. Quoted "$ENV_QEMU_TIMEOUT" in the sleep call (line ~196).
All ENV_* variables are already validated against strict regexes before reaching these command lines, providing defense-in-depth.


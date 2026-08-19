<!-- markdownlint-disable -->

# Hardening Report: tobozo--esp32-qemu-sim/v2.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tobozo--esp32-qemu-sim/v2.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple `uses:` references in action.yml and workflow files use mutable tags or branch names instead of pinned 40-character SHA digests, making the action vulnerable to supply-chain attacks if the referenced tag is moved or the branch is updated.

In action.yml:
- `uses: actions/cache@v3` (appears twice)
- `uses: actions/checkout@v3` (appears twice)
- `uses: actions/setup-python@v5.4.0`
- `uses: actions/upload-artifact@v4`

In .github/workflows/test-esp32.yml:
- `uses: actions/checkout@v3`
- `uses: ArminJo/arduino-test-compile@v3.2.0`
- `uses: tobozo/esp32-qemu-sim@main`

In .github/workflows/test-esp32c3.yml:
- `uses: actions/checkout@v3`
- `uses: ArminJo/arduino-test-compile@v3.2.0`
- `uses: tobozo/esp32-qemu-sim@main`

In .github/workflows/test-esp32s3.yml:
- `uses: actions/checkout@v3`
- `uses: ArminJo/arduino-test-compile@v3.2.0`
- `uses: tobozo/esp32-qemu-sim@main`

Locations:

- `action.yml:68`
- `action.yml:76`
- `action.yml:96`
- `action.yml:103`
- `action.yml:115`
- `action.yml:155`
- `.github/workflows/test-esp32.yml:16`
- `.github/workflows/test-esp32.yml:20`
- `.github/workflows/test-esp32.yml:34`
- `.github/workflows/test-esp32c3.yml:16`
- `.github/workflows/test-esp32c3.yml:20`
- `.github/workflows/test-esp32c3.yml:34`
- `.github/workflows/test-esp32s3.yml:16`
- `.github/workflows/test-esp32s3.yml:20`
- `.github/workflows/test-esp32s3.yml:35`

### script-injection (severity: high)

Sub-rule (a) violation: In the 'Run Build in QEmu' step of action.yml, the `run:` value directly interpolates a `${{ }}` expression: `run: ${{ github.action_path }}/run-build-in-qemu.sh`. Although `github.action_path` is not attacker-controlled in the same way as `github.head_ref`, any `${{ ... }}` expression inside a `run:` shell command string undergoes YAML template substitution before the shell processes it, which is a script-injection pattern. The safe alternative is to use the `$GITHUB_ACTION_PATH` environment variable instead.

Locations:

- `action.yml:148`

### missing-permissions (severity: medium)

None of the three workflow files define a top-level `permissions:` key, and no job within any of them defines a job-level `permissions:` key. Without explicit permissions, workflows run with the repository's default token permissions, which may be overly broad (e.g., write access to contents). Each workflow file should declare minimal required permissions.

Locations:

- `.github/workflows/test-esp32.yml:1`
- `.github/workflows/test-esp32c3.yml:1`
- `.github/workflows/test-esp32s3.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings: (1) Pinned all 9 unique action references across action.yml and 3 workflow files to full 40-char SHAs using lookup_action_sha, preserving original tags as comments. (2) Replaced `${{ github.action_path }}` template expression in the run: shell command with the built-in `$GITHUB_ACTION_PATH` environment variable to eliminate script-injection risk. (3) Added `permissions: contents: read` top-level block to all three workflow files (test-esp32.yml, test-esp32c3.yml, test-esp32s3.yml) — the minimal permission needed for checkout operations.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed all unquoted $ENV_* variable expansions in run-build-in-qemu.sh:
1. Line ~97 (csvdata): Replaced backtick with $() and quoted $ENV_BUILD_FOLDER/$ENV_PARTITIONS_CSV.
2. Line ~121 (esptool command): Quoted $ESPTOOL, $ENV_CHIP, ${ENV_FLASH_SIZE}MB, $ENV_BOOTLOADER_ADDR, $ENV_BUILD_FOLDER/$ENV_BOOTLOADER_BIN, $ENV_PARTITIONS_ADDR, $ENV_BUILD_FOLDER/$ENV_PARTITIONS_BIN, $OTADATA_ADDR, $ENV_BUILD_FOLDER/$ENV_OTADATA_BIN, $FIRMWARE_ADDR, $ENV_BUILD_FOLDER/$ENV_FIRMWARE_BIN, $SPIFFS_ADDR, $ENV_BUILD_FOLDER/$ENV_SPIFFS_BIN.
3. Line ~130 (QEMU command): Quoted $QEMU_BIN and $ENV_CHIP; replaced $ENV_PSRAM (which was set to '-m value' as a string) with a bash array PSRAM_ARGS=(-m "$ENV_PSRAM") to keep flag and value as separate arguments; quoted $log_file in tee.
4. Line ~143 (sleep): Quoted $ENV_QEMU_TIMEOUT.
5. Additional: Fixed timeout=$ENV_QEMU_TIMEOUT assignment, sleep $interval, tail ${log_file} backtick, grep_result backtick, and killall backtick.


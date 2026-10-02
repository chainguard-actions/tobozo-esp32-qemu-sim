<!-- markdownlint-disable -->

# Hardening Report: tobozo--esp32-qemu-sim/v2.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tobozo--esp32-qemu-sim/v2.1.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All 6 `uses:` references in action.yml use mutable version tags instead of pinned 40-character SHA commit hashes, making the action vulnerable to supply-chain attacks if any upstream action is compromised or its tag is moved.

Failing references:
- `uses: actions/cache@v3` (Cache QEmu build step)
- `uses: actions/checkout@v3` (Checkout qemu-xtensa step)
- `uses: actions/setup-python@v5.4.0` (Setup Python 3.11 step)
- `uses: actions/cache@v3` (Cache pip step)
- `uses: actions/checkout@v3` (Checkout esptool.py step)
- `uses: actions/upload-artifact@v4` (Upload logs as artifact step)

Locations:

- `action.yml:79`
- `action.yml:87`
- `action.yml:99`
- `action.yml:104`
- `action.yml:114`
- `action.yml:140`

### script-injection (severity: high)

Sub-rule (a): A `${{ }}` expression is directly interpolated inside a `run:` shell command string. The line `run: ${{ github.action_path }}/run-build-in-qemu.sh` embeds `${{ github.action_path }}` directly in the run value. Any `${{ ... }}` inside a run: block is a script-injection risk because the value is substituted by the YAML template engine before the shell ever sees it, allowing an attacker-controlled value to break out of the intended command.

Locations:

- `action.yml:136`

### script-injection (severity: high)

Sub-rule (b): Multiple env vars that hold values sourced from `inputs.*` (set via the `env:` block in action.yml) are used unquoted in shell commands inside run-build-in-qemu.sh. Unquoted shell variable expansions allow shell metacharacters (`;`, `|`, `&`, `$(...)`, whitespace, glob chars) embedded in the input values to be interpreted by the shell, enabling command injection.

Examples of unquoted expansions of workflow-controllable variables:
- Line 130: `$ESPTOOL --chip $ENV_CHIP merge-bin --fill-flash-size ${ENV_FLASH_SIZE}MB -o $QEMU_FLASH_IMAGE ...` (ENV_CHIP, ENV_FLASH_SIZE, QEMU_FLASH_IMAGE, ENV_BUILD_FOLDER, etc. all unquoted)
- Line 155: `($QEMU_BIN -nographic -machine $ENV_CHIP $ENV_PSRAM -drive file=$QEMU_FLASH_IMAGE,...)` (QEMU_BIN, ENV_CHIP, ENV_PSRAM, QEMU_FLASH_IMAGE all unquoted)

All of these variables are populated from `inputs.*` values supplied by the calling workflow and must be double-quoted (`"$VAR"`) in every shell expansion.

Locations:

- `run-build-in-qemu.sh:130`
- `run-build-in-qemu.sh:155`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all 6 unpinned uses references in action.yml with full SHA hashes (actions/cache@v3 x2, actions/checkout@v3 x2, actions/setup-python@v5.4.0, actions/upload-artifact@v4). Fixed script-injection (a) by moving github.action_path out of run: into the env: block as ACTION_PATH. Fixed script-injection (b) in run-build-in-qemu.sh by: (1) converting ENV_PSRAM from a string '-m VALUE' to a bash array PSRAM_ARGS to safely handle the two-token case, (2) quoting all input-derived variables in the esptool merge-bin command, (3) quoting all variables in the QEMU run command using the PSRAM_ARGS array, and (4) fixing the csvdata backtick command to use $() with quoted path variables.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two unquoted uses of $ENV_QEMU_TIMEOUT in run-build-in-qemu.sh: (1) `timeout=$ENV_QEMU_TIMEOUT` → `timeout="$ENV_QEMU_TIMEOUT"` (line ~218), and (2) `sleep $ENV_QEMU_TIMEOUT` → `sleep "$ENV_QEMU_TIMEOUT"` (line ~228). Also fixed `sleep $interval` → `sleep "$interval"` for consistency. Although a numeric regex validation is applied earlier in the script, double-quoting at the point of use is required to prevent shell metacharacter interpretation.


<!-- markdownlint-disable -->

# Hardening Report: tobozo--esp32-qemu-sim/v2.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **tobozo--esp32-qemu-sim/v2.0.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All 6 'uses:' references in action.yml are pinned to mutable tags or version strings rather than immutable 40-character SHA digests. This exposes the action to supply-chain attacks if any upstream action is compromised or its tag is moved. Failing references: actions/cache@v3 (×2), actions/checkout@v3 (×2), actions/setup-python@v5.4.0, actions/upload-artifact@v4.

Locations:

- `action.yml:70`
- `action.yml:79`
- `action.yml:92`
- `action.yml:98`
- `action.yml:108`
- `action.yml:134`

### script-injection (severity: high)

Rule (a): The 'run:' block in the 'Run Build in QEmu' step directly interpolates the GitHub Actions expression '${{ github.action_path }}' into the shell command string: `run: ${{ github.action_path }}/run-build-in-qemu.sh`. Any ${{ ... }} expression inside a run: block is a script-injection risk because the value is substituted by the YAML template engine before the shell ever sees it. Rule (b): The shell script run-build-in-qemu.sh uses numerous $ENV_* variables (populated from user-controlled inputs.* values) without double-quoting in shell commands. For example: `$ESPTOOL --chip $ENV_CHIP merge-bin --fill-flash-size ${ENV_FLASH_SIZE}MB ...` and `$QEMU_BIN -nographic -machine $ENV_CHIP $ENV_PSRAM ...` and `cat $ENV_BUILD_FOLDER/$ENV_PARTITIONS_CSV`. Unquoted expansions allow shell metacharacters in input values (semicolons, pipes, backticks, etc.) to be interpreted by the shell, enabling command injection.

Locations:

- `action.yml:130`
- `run-build-in-qemu.sh:131`
- `run-build-in-qemu.sh:155`
- `run-build-in-qemu.sh:161`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all 6 unpinned 'uses:' references in action.yml by pinning to full 40-char SHA digests (actions/cache@v3 ×2, actions/checkout@v3 ×2, actions/setup-python@v5.4.0, actions/upload-artifact@v4). Fixed script injection in action.yml by moving ${{ github.action_path }} into the env: block as ACTION_PATH and referencing it as "$ACTION_PATH/run-build-in-qemu.sh" in the run: command. Fixed unquoted variable expansions in run-build-in-qemu.sh: quoted the cat command path (line 131), quoted all arguments in the esptool merge-bin command (line 155), and quoted QEMU_BIN, ENV_CHIP, and log_file in the QEMU launch command (line 161). ENV_PSRAM is intentionally left unquoted as it's either empty or '-m 4M' requiring word-splitting, and is already validated by regex.


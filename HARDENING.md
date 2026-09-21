<!-- markdownlint-disable -->

# Hardening Report: kyverno--action-install-cli/v0.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **kyverno--action-install-cli/v0.1.0** was hardened automatically. 15 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The main `run:` block in action.yml directly interpolates multiple `${{ ... }}` expressions inside shell commands (sub-rule a). This causes GitHub Actions to perform YAML template substitution before the shell ever sees the value, allowing an attacker-controlled input to inject arbitrary shell commands. Affected expressions include:
- `${{ inputs.install-dir }}` used in `mkdir -p`, `pushd`, and `ln -s` (lines 39, 46, 50)
- `${{ inputs.release }}` used in `if [[ ... == "main" ]]`, regex match, string concatenation, and URL construction (lines 42, 98, 99, 101, 104, 105)
- `${{ inputs.use-sudo }}` used in `if [[ "${{ inputs.use-sudo }}" == "true" ]]` (line 94)
- `${{ runner.os }}` and `${{ runner.arch }}` used in unquoted `case` statements (lines 52, 53, 62, 66, 75, 79, 85, 90, 110, 121)
Additionally, the PATH-appending steps directly interpolate `${{ inputs.install-dir }}` in their `run:` commands (lines 128, 130). All `${{ ... }}` expressions must be moved to `env:` variables and the shell expansions must be double-quoted.

Locations:

- `action.yml:39`
- `action.yml:42`
- `action.yml:46`
- `action.yml:50`
- `action.yml:52`
- `action.yml:53`
- `action.yml:94`
- `action.yml:98`
- `action.yml:104`
- `action.yml:105`
- `action.yml:128`
- `action.yml:130`

### github-env-injection (severity: high)

Two steps write the untrusted `inputs.install-dir` value directly to `$GITHUB_PATH` without the required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`). An attacker-controlled `install-dir` input containing newlines could inject arbitrary entries into `$GITHUB_PATH`, potentially hijacking subsequent PATH lookups.
- Line 128 (bash): `echo "${{ inputs.install-dir }}" >> $GITHUB_PATH`
- Line 130 (pwsh): `echo "${{ inputs.install-dir }}" | Out-File -FilePath $env:GITHUB_PATH -Encoding utf8 -Append`
The value must be sanitized before being written to the special environment file.

Locations:

- `action.yml:128`
- `action.yml:130`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.install-dir }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:41`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:44`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.install-dir}}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:48`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.install-dir }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:54`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.use-sudo }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:111`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:116`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:117`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:119`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:123`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:124`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:126`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.install-dir }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:156`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.install-dir }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:159`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

Rewrote action.yml to move all ${{ inputs.install-dir }}, ${{ inputs.release }}, ${{ inputs.use-sudo }}, ${{ runner.os }}, and ${{ runner.arch }} expressions from run: blocks into env: blocks (as INPUT_INSTALL_DIR, INPUT_RELEASE, INPUT_USE_SUDO, RUNNER_OS, RUNNER_ARCH). All shell references are now double-quoted. The two $GITHUB_PATH-writing steps now sanitize INPUT_INSTALL_DIR with 'printf | tr -d' (bash) and '-replace' (pwsh) to strip newlines before writing, fixing the github-env-injection findings. No ${{ }} expressions remain in any run: block.


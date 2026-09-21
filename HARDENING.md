<!-- markdownlint-disable -->

# Hardening Report: kyverno--action-install-cli/v0.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **kyverno--action-install-cli/v0.2.0** was hardened automatically. 17 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The composite action's main `run:` block directly interpolates multiple `${{ inputs.* }}` and `${{ runner.* }}` expressions into shell commands without routing them through env vars. This means attacker-controlled values (e.g. `inputs.release`, `inputs.install-dir`, `inputs.use-sudo`, `inputs.verify`) are substituted into the shell script by the Actions runner before the shell ever parses the string, enabling command injection. Violating lines include:
- Line 43: `mkdir -p ${{ inputs.install-dir }}`
- Line 46: `if [[ ${{ inputs.release }} == "main" ]]`
- Line 50: `ln -s $GOBIN/kubectl-kyverno ${{ inputs.install-dir}}/kyverno`
- Line 54: `pushd ${{ inputs.install-dir }}`
- Line 56: `case ${{ runner.os }} in` (and repeated at line 133)
- Lines 57, 71, 85: `case ${{ runner.arch }} in`
- Lines 66, 80, 92: `log_error "unsupported architecture ${{ runner.arch }}"`
- Lines 97, 143: `log_error "unsupported os ${{ runner.os }}"`
- Line 101: `if [[ "${{ inputs.use-sudo }}" == "true" ]]`
- Lines 105–108: `if [[ ${{ inputs.release }} =~ $semver ]]` / log messages with `${{ inputs.release }}`
- Lines 112–113: `release_archive=kyverno-cli_${{ inputs.release }}_...` / `release_archive_url=.../${{ inputs.release }}/...`
- Line 118: `if [[ "${{ inputs.verify }}" == "true" ]]`
- Line 126: `--certificate-identity=...@refs/tags/${{ inputs.release }}`
- Line 149: `echo "${{ inputs.install-dir }}" >> $GITHUB_PATH`
- Line 151: `echo "${{ inputs.install-dir }}" | Out-File -FilePath $env:GITHUB_PATH`
All `${{ ... }}` expressions must be moved to `env:` variables and the shell expansions must be double-quoted.

Locations:

- `action.yml:43`
- `action.yml:46`
- `action.yml:50`
- `action.yml:54`
- `action.yml:56`
- `action.yml:57`
- `action.yml:66`
- `action.yml:71`
- `action.yml:80`
- `action.yml:85`
- `action.yml:92`
- `action.yml:97`
- `action.yml:101`
- `action.yml:105`
- `action.yml:106`
- `action.yml:108`
- `action.yml:112`
- `action.yml:113`
- `action.yml:118`
- `action.yml:126`
- `action.yml:133`
- `action.yml:143`
- `action.yml:149`
- `action.yml:151`

### github-env-injection (severity: high)

Two steps write the untrusted input `${{ inputs.install-dir }}` directly to `$GITHUB_PATH` (and `$env:GITHUB_PATH` in PowerShell) without first sanitizing the value with `printf '%s' ... | tr -d '\n\r'`. A caller can supply a newline-containing value to inject arbitrary entries into `$GITHUB_PATH`, potentially hijacking PATH-based binary resolution for subsequent steps. The bash step at line 149 (`echo "${{ inputs.install-dir }}" >> $GITHUB_PATH`) and the PowerShell step at line 151 (`echo "${{ inputs.install-dir }}" | Out-File -FilePath $env:GITHUB_PATH`) are both affected.

Locations:

- `action.yml:149`
- `action.yml:151`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.install-dir }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:45`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:48`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.install-dir}}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:52`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.install-dir }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:58`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.use-sudo }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:115`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:120`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:121`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:123`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:127`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:128`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:130`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.verify }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:133`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.release }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:140`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.install-dir }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:174`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.install-dir }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:177`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

Rewrote hardened/action/action.yml to fix all injection findings:
1. Added an env: block to the main bash step mapping all ${{ inputs.* }} and ${{ runner.* }} expressions to environment variables (INPUT_RELEASE, INPUT_INSTALL_DIR, INPUT_USE_SUDO, INPUT_VERIFY, RUNNER_OS, RUNNER_ARCH).
2. Replaced all inline ${{ }} expressions in the run: block with properly double-quoted $ENV_VAR references.
3. Fixed the GITHUB_PATH injection: the bash step now sanitizes INPUT_INSTALL_DIR with `printf '%s' "$INPUT_INSTALL_DIR" | tr -d '\n\r'` before writing to $GITHUB_PATH; the PowerShell step uses `-replace` to strip newlines.
4. All remaining ${{ }} expressions are only in env: blocks and if: conditions, which are evaluated by the Actions runner and not passed to the shell.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted expansions of ${INPUT_RELEASE} in two variable assignments in action.yml (lines 97-98). Changed:
  release_archive=kyverno-cli_${INPUT_RELEASE}_${release_archive}
  release_archive_url=https://github.com/kyverno/kyverno/releases/download/${INPUT_RELEASE}/${release_archive}
to:
  release_archive="kyverno-cli_${INPUT_RELEASE}_${release_archive}"
  release_archive_url="https://github.com/kyverno/kyverno/releases/download/${INPUT_RELEASE}/${release_archive}"
The entire right-hand side of each assignment is now double-quoted, ensuring the user-controlled INPUT_RELEASE value cannot be interpreted as shell metacharacters.


<!-- markdownlint-disable -->

# Hardening Report: pyvista--setup-headless-display-action/v4.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **pyvista--setup-headless-display-action/v4.1** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Three `${{ }}` expressions are interpolated directly inside `run:` shell command strings in action.yml, before the shell ever sees the value. (1) Line 79: `if [ "${{ inputs.mesa3d-release }}" == "latest" ]` — attacker-controlled input interpolated into a shell comparison. (2) Line 83: `export MESA3D_VERSION=${{ inputs.mesa3d-release }}` — unquoted attacker-controlled input interpolated directly into a shell assignment, allowing shell metacharacter injection. (3) Line 91: `run: bash ${{ github.action_path }}\windows\install_opengl.sh` — `github.action_path` interpolated directly into a `cmd` shell command string. All three must be replaced with env-var references (e.g. set via `env:` and referenced as `"$VAR"` in the script).

Locations:

- `action.yml:79`
- `action.yml:83`
- `action.yml:91`

### github-env-injection (severity: high)

The `MESA3D_VERSION` shell variable is derived from the untrusted input `${{ inputs.mesa3d-release }}` (lines 79/83) and then written to `$GITHUB_ENV` on line 85 via `echo "MESA3D_VERSION=${MESA3D_VERSION}" | tee -a $GITHUB_ENV` without the required sanitization step (`printf '%s' "$MESA3D_VERSION" | tr -d '\n\r'`). An attacker can supply a newline-containing value to inject arbitrary environment variables into subsequent steps.

Locations:

- `action.yml:85`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.mesa3d-release }}" appears directly in run: block of step "Determine OpenGL version to install on Windows"; move to env: map

Locations:

- `action.yml:85`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.mesa3d-release }}" appears directly in run: block of step "Determine OpenGL version to install on Windows"; move to env: map

Locations:

- `action.yml:90`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

Fixed all four findings in action.yml:
1. Moved `${{ inputs.mesa3d-release }}` to an `env:` block as `MESA3D_RELEASE` and referenced it as `"$MESA3D_RELEASE"` in the bash shell (lines 79 and 83).
2. Added sanitization of `MESA3D_VERSION` via `printf '%s' "$MESA3D_VERSION" | tr -d '\n\r'` before writing to `$GITHUB_ENV` (line 85).
3. Moved `${{ github.action_path }}` to an `env:` block as `ACTION_PATH` and referenced it as `"%ACTION_PATH%"` in the `cmd` shell command (line 91), using Windows cmd variable expansion syntax.

### Iteration 1

**Fixes applied:** publish-gate

**Notes:**

Added `|| true` to the MESA3D_VERSION=$(curl ...) assignment in action.yml line 89. This preserves upstream's behavior where a non-zero exit from curl (or the piped grep/sed) doesn't abort the step under bash's errexit mode, matching the behavior upstream had when the command was run inside echo/printf's argument substitution.


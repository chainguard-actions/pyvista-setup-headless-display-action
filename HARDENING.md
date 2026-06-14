<!-- markdownlint-disable -->

# Hardening Report: pyvista--setup-headless-display-action/v4.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **pyvista--setup-headless-display-action/v4.2** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple ${{ ... }} expressions are interpolated directly inside run: shell command strings in action.yml, violating rule (a). (1) Line 82: `if [ "${{ inputs.mesa3d-release }}" == "latest" ]` — attacker-controlled input interpolated into a shell condition. (2) Line 83: `if [[ "${{ inputs.install-mesa3d-offscreen }}" == "true" ]]` — attacker-controlled input interpolated into a shell condition. (3) Line 90: `export MESA3D_VERSION=${{ inputs.mesa3d-release }}` — unquoted attacker-controlled input interpolated directly into a shell variable assignment, allowing shell metacharacter injection. (4) Line 101: `run: bash ${{ github.action_path }}\windows\install_opengl.sh` — github.action_path interpolated directly into a shell command string.

Locations:

- `action.yml:82`
- `action.yml:83`
- `action.yml:90`
- `action.yml:101`

### github-env-injection (severity: high)

In the 'Determine OpenGL version to install on Windows' step, the variable MESA3D_VERSION is set directly from the attacker-controlled input `${{ inputs.mesa3d-release }}` (line 90: `export MESA3D_VERSION=${{ inputs.mesa3d-release }}`), then written to $GITHUB_ENV without sanitization (line 92: `echo "MESA3D_VERSION=${MESA3D_VERSION}" | tee -a $GITHUB_ENV`). An attacker can inject newlines into the input to poison GITHUB_ENV with arbitrary environment variable assignments. The required sanitization step (`printf '%s' "$MESA3D_VERSION" | tr -d '\n\r'`) is absent.

Locations:

- `action.yml:92`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.mesa3d-release }}" appears directly in run: block of step "Determine OpenGL version to install on Windows"; move to env: map

Locations:

- `action.yml:85`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.install-mesa3d-offscreen }}" appears directly in run: block of step "Determine OpenGL version to install on Windows"; move to env: map

Locations:

- `action.yml:86`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.mesa3d-release }}" appears directly in run: block of step "Determine OpenGL version to install on Windows"; move to env: map

Locations:

- `action.yml:95`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

Fixed all findings in action.yml:
1. 'Determine OpenGL version to install on Windows' step: Added env: block with MESA3D_RELEASE and INSTALL_MESA3D_OFFSCREEN variables, replacing all ${{ inputs.mesa3d-release }} and ${{ inputs.install-mesa3d-offscreen }} inline expressions. Added newline sanitization (`safe=$(printf '%s' "$MESA3D_VERSION" | tr -d '\n\r')`) before writing to $GITHUB_ENV.
2. 'Install OpenGL on Windows' step: Moved ${{ github.action_path }} into an env: block as ACTION_PATH, changed shell from cmd to bash, and referenced it as "$ACTION_PATH/windows/install_opengl.sh".


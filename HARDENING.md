<!-- markdownlint-disable -->

# Hardening Report: pyvista--setup-headless-display-action/v4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **pyvista--setup-headless-display-action/v4** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): GitHub Actions expressions are interpolated directly inside `run:` shell command strings in the 'Determine OpenGL version to install on Windows' step. The expressions `${{ inputs.mesa3d-release }}` and `${{ inputs.install-mesa3d-offscreen }}` are substituted into the shell script before the shell parses it, allowing an attacker-controlled input to inject arbitrary shell commands. Offending lines:
  - `if [ "${{ inputs.mesa3d-release }}" == "latest" ]; then`
  - `if [[ "${{ inputs.install-mesa3d-offscreen }}" == "true" ]]; then`
  - `export MESA3D_VERSION=${{ inputs.mesa3d-release }}`
These inputs should be passed via `env:` variables and referenced as quoted shell variables (e.g., `"$MESA3D_RELEASE"`) instead.

Locations:

- `action.yml:79`
- `action.yml:80`
- `action.yml:87`

### script-injection (severity: high)

Sub-rule (a): The expression `${{ github.action_path }}` is interpolated directly inside a `run:` shell command string in the 'Install OpenGL on Windows' step: `run: bash ${{ github.action_path }}\windows\install_opengl.sh`. Even though `github.action_path` is GitHub-controlled, any `${{ ... }}` expression inside a `run:` block is a script-injection risk because the value is substituted into the shell script before the shell parses it. It should be referenced via the `$GITHUB_ACTION_PATH` environment variable instead.

Locations:

- `action.yml:101`

### github-env-injection (severity: high)

In the 'Determine OpenGL version to install on Windows' step, the shell variable `MESA3D_VERSION` is derived from the untrusted input `${{ inputs.mesa3d-release }}` and then written to `$GITHUB_ENV` without sanitization: `echo "MESA3D_VERSION=${MESA3D_VERSION}" | tee -a $GITHUB_ENV`. An attacker-controlled value containing newlines could inject additional environment variable assignments (e.g., `MESA3D_VERSION=foo\nPATH=/evil/bin:$PATH`). The required sanitization step (`printf '%s' "$MESA3D_VERSION" | tr -d '\n\r'`) must be applied before the write.

Locations:

- `action.yml:91`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.mesa3d-release }}" appears directly in run: block of step "Determine OpenGL version to install on Windows"; move to env: map

Locations:

- `action.yml:86`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.install-mesa3d-offscreen }}" appears directly in run: block of step "Determine OpenGL version to install on Windows"; move to env: map

Locations:

- `action.yml:87`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.mesa3d-release }}" appears directly in run: block of step "Determine OpenGL version to install on Windows"; move to env: map

Locations:

- `action.yml:96`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

Fixed all findings in action.yml:
1. 'Determine OpenGL version to install on Windows' step: Moved `${{ inputs.mesa3d-release }}` and `${{ inputs.install-mesa3d-offscreen }}` into an `env:` block as `MESA3D_RELEASE` and `INSTALL_MESA3D_OFFSCREEN`. All shell references now use plain env vars (`$MESA3D_RELEASE`, `$INSTALL_MESA3D_OFFSCREEN`).
2. Added newline sanitization before writing MESA3D_VERSION to $GITHUB_ENV: `safe_version=$(printf '%s' "$MESA3D_VERSION" | tr -d '\n\r')` and write `safe_version` instead.
3. 'Install OpenGL on Windows' step: Replaced `bash ${{ github.action_path }}\windows\install_opengl.sh` (cmd shell with expression interpolation) with `bash "$GITHUB_ACTION_PATH/windows/install_opengl.sh"` (bash shell using the built-in env var).


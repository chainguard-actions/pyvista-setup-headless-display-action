<!-- markdownlint-disable -->

# Hardening Report: pyvista--setup-headless-display-action/v4.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **pyvista--setup-headless-display-action/v4.3** was hardened automatically. 5 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Determine OpenGL version to install on Windows' run: block directly interpolates GitHub Actions expressions inside shell commands. `${{ inputs.mesa3d-release }}` appears in an `if` condition (quoted but still template-substituted before the shell sees it) and — critically — unquoted in `export MESA3D_VERSION=${{ inputs.mesa3d-release }}`, allowing an attacker-controlled value to inject arbitrary shell commands. `${{ inputs.install-mesa3d-offscreen }}` is similarly interpolated. Sub-rule (a) also applies to the 'Install OpenGL on Windows' step which uses `bash ${{ github.action_path }}\windows\install_opengl.sh` — any expression inside a run: block is a script-injection risk regardless of context.

Locations:

- `action.yml:82`
- `action.yml:83`
- `action.yml:90`
- `action.yml:101`

### github-env-injection (severity: high)

The 'Determine OpenGL version to install on Windows' step writes `MESA3D_VERSION` to `$GITHUB_ENV` via `echo "MESA3D_VERSION=${MESA3D_VERSION}" | tee -a $GITHUB_ENV`. The value of `MESA3D_VERSION` is derived directly from the untrusted input `${{ inputs.mesa3d-release }}` (set via `export MESA3D_VERSION=${{ inputs.mesa3d-release }}`). No sanitization (`printf '%s' ... | tr -d '\n\r'`) is applied before the write, allowing an attacker to inject arbitrary environment variable definitions into `$GITHUB_ENV` by embedding newlines in the input value.

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
1. 'Determine OpenGL version to install on Windows' step: Added env: block with MESA3D_RELEASE and INSTALL_MESA3D_OFFSCREEN variables, replaced all ${{ inputs.* }} interpolations in the run: block with plain env var references ($MESA3D_RELEASE, $INSTALL_MESA3D_OFFSCREEN). Added newline sanitization (printf '%s' | tr -d '\n\r') before writing MESA3D_VERSION to $GITHUB_ENV.
2. 'Install OpenGL on Windows' step: Changed shell from cmd to bash, added env: block with ACTION_PATH=${{ github.action_path }}, replaced the inline ${{ github.action_path }} interpolation with the env var reference ${ACTION_PATH}.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted variable expansions in windows/install_opengl.sh:
1. curl URL now wrapped in double quotes: "https://.../${MESA3D_VERSION}/${NAME}.7z"
2. 7z arguments now quoted: "${NAME}.7z" and "./${NAME}"
3. rm -Rf argument now quoted: "${NAME}"
The cmd.exe lines already had ${NAME} inside double-quoted strings so were already safe. All uses of the attacker-controllable ${MESA3D_VERSION} and derived ${NAME} variables are now properly double-quoted in command arguments.


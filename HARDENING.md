<!-- markdownlint-disable -->

# Hardening Report: pyvista--setup-headless-display-action/v4.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **pyvista--setup-headless-display-action/v4.1** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Determine OpenGL version to install on Windows' step directly interpolates ${{ inputs.mesa3d-release }} inside the run: shell script — once in a shell if-condition (if [ "${{ inputs.mesa3d-release }}" == "latest" ]) and once completely unquoted (export MESA3D_VERSION=${{ inputs.mesa3d-release }}). An attacker-controlled input value is substituted into the shell command string before the shell parses it, enabling arbitrary command injection via shell metacharacters.

Locations:

- `action.yml:80`
- `action.yml:85`

### script-injection (severity: high)

Sub-rule (a): The 'Install OpenGL on Windows' step directly interpolates ${{ github.action_path }} inside the run: shell command string: bash ${{ github.action_path }}\windows\install_opengl.sh. Any ${{ ... }} expression interpolated directly into a run: block is a script-injection risk as the value is substituted before the shell parses the command.

Locations:

- `action.yml:99`

### github-env-injection (severity: high)

The 'Determine OpenGL version to install on Windows' step writes MESA3D_VERSION — derived directly from the attacker-controllable input ${{ inputs.mesa3d-release }} — to $GITHUB_ENV via echo "MESA3D_VERSION=${MESA3D_VERSION}" | tee -a $GITHUB_ENV without the required sanitization step (printf '%s' "$MESA3D_VERSION" | tr -d '\n\r'). A newline-containing input value could inject arbitrary environment variables into subsequent steps.

Locations:

- `action.yml:87`

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

Fixed all findings in action.yml:
1. 'Determine OpenGL version to install on Windows' step: moved ${{ inputs.mesa3d-release }} to env block as MESA3D_RELEASE; replaced all inline ${{ inputs.mesa3d-release }} expressions in the run: block with $MESA3D_RELEASE; added newline sanitization (printf '%s' "$MESA3D_VERSION" | tr -d '\n\r') before writing to $GITHUB_ENV.
2. 'Install OpenGL on Windows' step: moved ${{ github.action_path }} to env block as ACTION_PATH; changed shell from cmd to bash; replaced inline expression with "${ACTION_PATH}/windows/install_opengl.sh".


<!-- markdownlint-disable -->

# Hardening Report: pyvista--setup-headless-display-action/v5.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **pyvista--setup-headless-display-action/v5.2.0** was hardened automatically. 7 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): the step 'Determine OpenGL version to install on Windows' puts input expressions straight into its run: script: `if [ "${{ inputs.mesa3d-release }}" == "latest" ]`, `if [[ "${{ inputs.install-mesa3d-offscreen }}" == "true" ]]`, and `export MESA3D_VERSION=${{ inputs.mesa3d-release }}` (unquoted). A crafted input could run arbitrary shell commands.

Locations:

- `action.yml:121`
- `action.yml:122`
- `action.yml:131`

### script-injection (severity: high)

Rule (a): the step 'Install OpenGL on Windows' puts an expression straight into its run: command: `run: bash ${{ github.action_path }}\windows\install_opengl.sh`. Read the path from an env var instead, and quote it.

Locations:

- `action.yml:146`

### github-env-injection (severity: high)

The step 'Determine OpenGL version to install on Windows' builds MESA3D_VERSION from inputs.mesa3d-release, or from an unauthenticated curl response. It then writes the value to $GITHUB_ENV without sanitizing it first: `echo "MESA3D_VERSION=${MESA3D_VERSION}" | tee -a $GITHUB_ENV`. There is no printf '%s' ... | tr -d '\n\r' step, so a value containing a newline could add arbitrary environment variables.

Locations:

- `action.yml:133`

### github-env-injection (severity: high)

The step 'Start a headless Wayland compositor (Weston) on Linux' writes `echo "XDG_RUNTIME_DIR=$XDG_RUNTIME_DIR"` and `echo "WESTON_LOG=$log"` to $GITHUB_ENV without sanitizing them. XDG_RUNTIME_DIR comes from `${XDG_RUNTIME_DIR:-...}`, which is inherited from the calling workflow's environment, and log is built from the inherited RUNNER_TEMP. Neither goes through printf '%s' ... | tr -d '\n\r' before the write.

Locations:

- `action.yml:215`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.mesa3d-release }}" appears directly in run: block of step "Determine OpenGL version to install on Windows"; move to env: map

Locations:

- `action.yml:121`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.install-mesa3d-offscreen }}" appears directly in run: block of step "Determine OpenGL version to install on Windows"; move to env: map

Locations:

- `action.yml:122`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.mesa3d-release }}" appears directly in run: block of step "Determine OpenGL version to install on Windows"; move to env: map

Locations:

- `action.yml:131`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

All changes are in action.yml. Nothing was run or tested.

1. Windows 'Determine OpenGL version' step: the mesa3d-release and install-mesa3d-offscreen inputs now come in through env: (INPUT_MESA3D_RELEASE and INPUT_INSTALL_MESA3D_OFFSCREEN). The script reads them as quoted variables. MESA3D_VERSION has newlines stripped with printf/tr before it is written to $GITHUB_ENV.

2. Windows 'Install OpenGL' step (cmd shell): the action path now comes from env as ACTION_PATH. The step runs bash "%ACTION_PATH%\windows\install_opengl.sh", using cmd's %VAR% syntax with the path quoted.

3. Weston step: XDG_RUNTIME_DIR and the log path have newlines stripped before they are written to $GITHUB_ENV.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

In action.yml, the Weston step now writes WESTON_LOG=$safe_log to $GITHUB_ENV. Before, it wrote the raw $log value, which is built from RUNNER_TEMP and still had any newlines in it.


<!-- markdownlint-disable -->

# Hardening Report: pyvista--setup-headless-display-action/v5.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **pyvista--setup-headless-display-action/v5.1.0** was hardened automatically. 6 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple `${{ inputs.* }}` expressions are interpolated directly inside `run:` shell command strings in the 'Determine OpenGL version to install on Windows' step, allowing an attacker-controlled value to be injected into the shell before quoting can protect it.

Offending lines:
- Line 120: `if [ "${{ inputs.mesa3d-release }}" == "latest" ]; then`
- Line 121: `if [[ "${{ inputs.install-mesa3d-offscreen }}" == "true" ]]; then`
- Line 130: `export MESA3D_VERSION=${{ inputs.mesa3d-release }}` (also sub-rule (b): unquoted expansion)

Fix: move inputs into `env:` variables and reference them as double-quoted shell variables (`"$MESA3D_RELEASE"`, `"$INSTALL_MESA3D_OFFSCREEN"`) inside the `run:` block.

Locations:

- `action.yml:120`
- `action.yml:121`
- `action.yml:130`

### script-injection (severity: high)

Sub-rule (a): `${{ github.action_path }}` is interpolated directly inside a `run:` shell command string in the 'Install OpenGL on Windows' step. Although `github.action_path` is GitHub-controlled, any `${{ ... }}` expression inside a `run:` block is a script-injection risk because the value flows through YAML template substitution before the shell ever sees it.

Offending line:
- Line 146: `run: bash ${{ github.action_path }}\windows\install_opengl.sh`

Fix: use the `$GITHUB_ACTION_PATH` environment variable instead: `run: bash "$GITHUB_ACTION_PATH/windows/install_opengl.sh"`

Locations:

- `action.yml:146`

### github-env-injection (severity: high)

In the 'Determine OpenGL version to install on Windows' step, the value of `inputs.mesa3d-release` is interpolated directly into the shell variable `MESA3D_VERSION` (line 130: `export MESA3D_VERSION=${{ inputs.mesa3d-release }}`), and then that variable is written to `$GITHUB_ENV` on line 132 (`echo "MESA3D_VERSION=${MESA3D_VERSION}" | tee -a $GITHUB_ENV`) without the required sanitization step (`printf '%s' "$MESA3D_VERSION" | tr -d '\n\r'`). A newline embedded in the input value could inject arbitrary environment variables into subsequent steps.

Fix: sanitize before writing:
```
safe=$(printf '%s' "$MESA3D_VERSION" | tr -d '\n\r')
echo "MESA3D_VERSION=${safe}" >> "$GITHUB_ENV"
```

Locations:

- `action.yml:132`

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

Fixed all findings in action.yml:
1. 'Determine OpenGL version to install on Windows' step: Added `env:` block with `MESA3D_RELEASE: ${{ inputs.mesa3d-release }}` and `INSTALL_MESA3D_OFFSCREEN: ${{ inputs.install-mesa3d-offscreen }}`. Replaced all inline `${{ inputs.* }}` expressions in the `run:` block with `"$MESA3D_RELEASE"` and `"$INSTALL_MESA3D_OFFSCREEN"`. Added sanitization before writing to GITHUB_ENV: `safe=$(printf '%s' "$MESA3D_VERSION" | tr -d '\n\r')` and `echo "MESA3D_VERSION=${safe}" >> "$GITHUB_ENV"`.
2. 'Install OpenGL on Windows' step: Replaced `bash ${{ github.action_path }}\windows\install_opengl.sh` with `bash "$GITHUB_ACTION_PATH/windows/install_opengl.sh"` and changed shell from `cmd` to `bash` so the environment variable expands correctly.

### Iteration 2

**Fixes applied:** missing-permissions, unpinned-uses, script-injection

**Notes:**

Fixed all 7 findings across two workflow files:

**local.yml:**
- Added top-level `permissions: contents: read` block (missing-permissions)
- Pinned actions/checkout@v7, actions/setup-python@v7, and actions/upload-artifact@v7 (×3) to full SHA digests (unpinned-uses)
- Moved `${{ env.ARTIFACT_PREFIX }}` into step env block, referenced as `$ARTIFACT_PREFIX` in shell (script-injection)
- Moved `${{ matrix.qt }}` into step env block as `MATRIX_QT`, referenced as `"$MATRIX_QT"` in pip install (script-injection)
- Moved `${{ steps.pyvista.outcome }}` and `${{ steps.qt.outcome }}` into step env block as `PYVISTA_OUTCOME`/`QT_OUTCOME`, referenced as shell variables (script-injection)

**production.yml:**
- Pinned pyvista/setup-headless-display-action@v5, actions/setup-python@v7, and actions/upload-artifact@v7 to full SHA digests (unpinned-uses)
- Moved `${{ matrix.os }}` into step env block as `MATRIX_OS`, used `os.environ['MATRIX_OS']` in the Python one-liner to avoid shell injection (script-injection)


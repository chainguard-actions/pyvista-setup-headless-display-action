<!-- markdownlint-disable -->

# Hardening Report: pyvista--setup-headless-display-action/v3.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **pyvista--setup-headless-display-action/v3.3** was hardened automatically. 10 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): ${{ inputs.mesa3d-release }} is directly interpolated inside a run: shell command in two places — once in a quoted comparison (`if [ "${{ inputs.mesa3d-release }}" == "latest" ]`) and once unquoted (`export MESA3D_VERSION=${{ inputs.mesa3d-release }}`). Additionally, ${{ github.action_path }} is directly interpolated in a cmd run: block (`run: bash ${{ github.action_path }}\windows\install_opengl.sh`). All three are YAML-template-substituted before the shell sees them, enabling command injection via a crafted input value.

Locations:

- `action.yml:70`
- `action.yml:74`
- `action.yml:80`

### github-env-injection (severity: high)

The 'Determine OpenGL version to install on Windows' step writes MESA3D_VERSION to $GITHUB_ENV via `echo "MESA3D_VERSION=${MESA3D_VERSION}" | tee -a $GITHUB_ENV`. MESA3D_VERSION is derived directly from ${{ inputs.mesa3d-release }} (an attacker-controlled input) without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A newline-embedded value in the input could inject arbitrary environment variables into subsequent steps.

Locations:

- `action.yml:76`

### script-injection (severity: high)

Sub-rule (a): Multiple run: blocks in local.yml directly interpolate ${{ matrix.os }} and ${{ matrix.qt }} (workflow-controllable matrix values) into shell commands without routing through env: variables. Offending lines: `python -c "import pyvista;pyvista.Cube().plot(screenshot='${{ matrix.os }}-${{ matrix.qt }}-cube.png')"` and `pip install ${{ matrix.qt }} matplotlib qtpy`. These allow shell metacharacter injection via matrix values.

Locations:

- `.github/workflows/local.yml:48`
- `.github/workflows/local.yml:51`

### script-injection (severity: high)

Sub-rule (a): A run: block in production.yml directly interpolates ${{ matrix.os }} (a workflow-controllable matrix value) into a shell command: `python -c "import pyvista;pyvista.Sphere().plot(screenshot='${{ matrix.os }}-sphere.png')"`. This allows shell metacharacter injection via a crafted matrix value.

Locations:

- `.github/workflows/production.yml:19`

### unpinned-uses (severity: high)

Multiple uses: references in local.yml are pinned to mutable version tags rather than full 40-character commit SHAs, making them vulnerable to supply-chain attacks: `actions/checkout@v4`, `actions/setup-python@v5`, `actions/upload-artifact@v4` (appears three times).

Locations:

- `.github/workflows/local.yml:32`
- `.github/workflows/local.yml:38`
- `.github/workflows/local.yml:44`
- `.github/workflows/local.yml:55`
- `.github/workflows/local.yml:64`

### unpinned-uses (severity: high)

Multiple uses: references in production.yml are pinned to mutable version tags rather than full 40-character commit SHAs: `pyvista/setup-headless-display-action@v3`, `actions/setup-python@v5`, `actions/upload-artifact@v4`.

Locations:

- `.github/workflows/production.yml:14`
- `.github/workflows/production.yml:15`
- `.github/workflows/production.yml:20`

### missing-permissions (severity: medium)

local.yml has no top-level permissions: block and no job-level permissions: block on the 'test' job. Without explicit permissions, the workflow inherits the default repository permissions (which can be write-all in some configurations), violating the principle of least privilege.

Locations:

- `.github/workflows/local.yml:1`

### missing-permissions (severity: medium)

production.yml has no top-level permissions: block and no job-level permissions: block on the 'test' job. Without explicit permissions, the workflow inherits the default repository permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/production.yml:1`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.mesa3d-release }}" appears directly in run: block of step "Determine OpenGL version to install on Windows"; move to env: map

Locations:

- `action.yml:78`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.mesa3d-release }}" appears directly in run: block of step "Determine OpenGL version to install on Windows"; move to env: map

Locations:

- `action.yml:83`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions, static-inline-injection

**Notes:**

Fixed all 10 findings across action.yml, local.yml, and production.yml:

1. action.yml script-injection: Moved `${{ inputs.mesa3d-release }}` to env block as MESA3D_RELEASE; changed Windows cmd step to use bash shell with ACTION_PATH env var instead of interpolating `${{ github.action_path }}`.

2. action.yml github-env-injection: Added `safe=$(printf '%s' "$MESA3D_VERSION" | tr -d '\n\r')` before writing to GITHUB_ENV.

3. local.yml script-injection: Moved `${{ matrix.os }}` and `${{ matrix.qt }}` to env blocks (MATRIX_OS, MATRIX_QT) in both the 'Second test of PyVista' and 'Test Qt' steps.

4. production.yml script-injection: Moved `${{ matrix.os }}` to env block as MATRIX_OS in the 'Use PyVista' step.

5. local.yml unpinned-uses: Pinned actions/checkout@v4→SHA, actions/setup-python@v5→SHA, actions/upload-artifact@v4→SHA (3 occurrences).

6. production.yml unpinned-uses: Pinned pyvista/setup-headless-display-action@v3→SHA, actions/setup-python@v5→SHA, actions/upload-artifact@v4→SHA.

7. local.yml missing-permissions: Added `permissions: contents: read` at top level.

8. production.yml missing-permissions: Added `permissions: contents: read` at top level.


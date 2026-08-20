<!-- markdownlint-disable -->

# Hardening Report: pyvista--setup-headless-display-action/v5.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **pyvista--setup-headless-display-action/v5.0.0** was hardened automatically. 11 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Determine OpenGL version to install on Windows' step directly interpolates ${{ inputs.mesa3d-release }} and ${{ inputs.install-mesa3d-offscreen }} inside run: shell commands. These are user-controlled inputs that flow through YAML template substitution before the shell sees them, enabling command injection. Offending lines: `if [ "${{ inputs.mesa3d-release }}" == "latest" ]`, `if [[ "${{ inputs.install-mesa3d-offscreen }}" == "true" ]]`, and `export MESA3D_VERSION=${{ inputs.mesa3d-release }}` (the last one is also unquoted, violating sub-rule b). Fix: move inputs into env: vars and reference them as quoted shell variables.

Locations:

- `action.yml:115`
- `action.yml:116`
- `action.yml:124`

### script-injection (severity: high)

Sub-rule (a): The 'Install OpenGL on Windows' step directly interpolates ${{ github.action_path }} inside a run: shell command: `run: bash ${{ github.action_path }}\windows\install_opengl.sh`. Even though github.action_path is GitHub-controlled, any ${{ ... }} expression directly inside a run: block is a script-injection finding per the check rules. Fix: use the $GITHUB_ACTION_PATH environment variable instead.

Locations:

- `action.yml:136`

### github-env-injection (severity: high)

The 'Determine OpenGL version to install on Windows' step writes MESA3D_VERSION to $GITHUB_ENV without sanitization. MESA3D_VERSION is derived from ${{ inputs.mesa3d-release }}, a user-controlled input. The write `echo "MESA3D_VERSION=${MESA3D_VERSION}" | tee -a $GITHUB_ENV` does not apply the required `printf '%s' ... | tr -d '\n\r'` sanitization before writing to the special environment file, allowing newline injection to set arbitrary environment variables.

Locations:

- `action.yml:126`

### script-injection (severity: high)

Sub-rule (a): Multiple run: steps in local.yml directly interpolate ${{ }} expressions inside shell commands. (1) `pip install ${{ matrix.qt }} matplotlib qtpy` — matrix.qt is workflow-controllable and unquoted. (2) `pyvista.Cube().plot(screenshot='${{ env.ARTIFACT_PREFIX }}-cube.png')` — env.ARTIFACT_PREFIX is workflow-controllable. (3) `if [[ "${{ steps.pyvista.outcome }}" == "failure" ]]` and `if [[ "${{ steps.qt.outcome }}" == "failure" ]]` — steps outputs are workflow-controllable. All of these should be moved to env: vars and referenced as quoted shell variables.

Locations:

- `.github/workflows/local.yml:68`
- `.github/workflows/local.yml:63`
- `.github/workflows/local.yml:93`
- `.github/workflows/local.yml:97`

### script-injection (severity: high)

Sub-rule (a): The 'Use PyVista' run: step in production.yml directly interpolates ${{ matrix.os }} inside a shell command: `python -c "import pyvista;pyvista.Sphere().plot(screenshot='${{ matrix.os }}-sphere.png')"`. matrix.os is a workflow-controllable value that flows through YAML template substitution before the shell sees it. Fix: move matrix.os into an env: var and reference it as a quoted shell variable.

Locations:

- `.github/workflows/production.yml:21`

### unpinned-uses (severity: high)

All uses: references in local.yml use mutable version tags instead of pinned 40-character SHA commit hashes, making the workflow vulnerable to supply-chain attacks if those tags are moved. Unpinned references: `actions/checkout@v7`, `actions/setup-python@v7`, `actions/upload-artifact@v7` (appears three times).

Locations:

- `.github/workflows/local.yml:44`
- `.github/workflows/local.yml:49`
- `.github/workflows/local.yml:58`
- `.github/workflows/local.yml:72`
- `.github/workflows/local.yml:80`

### unpinned-uses (severity: high)

All uses: references in production.yml use mutable version tags instead of pinned 40-character SHA commit hashes. Unpinned references: `pyvista/setup-headless-display-action@v4`, `actions/setup-python@v7`, `actions/upload-artifact@v7`.

Locations:

- `.github/workflows/production.yml:16`
- `.github/workflows/production.yml:18`
- `.github/workflows/production.yml:23`

### missing-permissions (severity: medium)

The local.yml workflow file has no top-level `permissions:` key and no job-level `permissions:` key on the `test` job. Without explicit permissions, the workflow inherits the repository's default token permissions (which may be read/write), violating the principle of least privilege. A `permissions:` block with only the required scopes should be added.

Locations:

- `.github/workflows/local.yml:1`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.mesa3d-release }}" appears directly in run: block of step "Determine OpenGL version to install on Windows"; move to env: map

Locations:

- `action.yml:120`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.install-mesa3d-offscreen }}" appears directly in run: block of step "Determine OpenGL version to install on Windows"; move to env: map

Locations:

- `action.yml:121`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.mesa3d-release }}" appears directly in run: block of step "Determine OpenGL version to install on Windows"; move to env: map

Locations:

- `action.yml:130`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions, static-inline-injection

**Notes:**

Fixed all findings across action.yml, local.yml, and production.yml:

**action.yml:**
- Moved `${{ inputs.mesa3d-release }}` and `${{ inputs.install-mesa3d-offscreen }}` to env: vars (MESA3D_RELEASE, INSTALL_MESA3D_OFFSCREEN) in the 'Determine OpenGL version' step
- Added `printf '%s' ... | tr -d '\n\r'` sanitization before writing MESA3D_VERSION to $GITHUB_ENV
- Replaced `bash ${{ github.action_path }}\windows\install_opengl.sh` with `bash %GITHUB_ACTION_PATH%\windows\install_opengl.sh` (cmd env var syntax)

**local.yml:**
- Added `permissions: contents: read` at top level
- Pinned actions/checkout@v7 → SHA 3d3c42e5...
- Pinned actions/setup-python@v7 → SHA 5fda3b95...
- Pinned actions/upload-artifact@v7 (×3) → SHA 043fb46d...
- Moved `${{ env.ARTIFACT_PREFIX }}` to env: var in 'Second test of PyVista' step
- Moved `${{ matrix.qt }}` to env var MATRIX_QT with xargs-based tokenization for safe package list expansion
- Moved `${{ steps.pyvista.outcome }}` and `${{ steps.qt.outcome }}` to env vars in 'Diagnose' step

**production.yml:**
- Pinned pyvista/setup-headless-display-action@v4 → SHA 5bc8de3b...
- Pinned actions/setup-python@v7 → SHA 5fda3b95...
- Pinned actions/upload-artifact@v7 → SHA 043fb46d...
- Moved `${{ matrix.os }}` to env var MATRIX_OS and used Python's os.environ to read it safely


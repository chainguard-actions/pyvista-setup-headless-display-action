<!-- markdownlint-disable -->

# Hardening Report: pyvista--setup-headless-display-action/v4.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **pyvista--setup-headless-display-action/v4.1** was hardened automatically. 7 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation inside run: blocks. In action.yml, `${{ inputs.mesa3d-release }}` is interpolated twice directly in a shell run: block — once in a quoted comparison (`if [ "${{ inputs.mesa3d-release }}" == "latest" ]`) and once unquoted (`export MESA3D_VERSION=${{ inputs.mesa3d-release }}`). Additionally, `${{ github.action_path }}` is interpolated directly in a run: block (`run: bash ${{ github.action_path }}\windows\install_opengl.sh`). Any of these allow an attacker-controlled value to be parsed by the shell before quoting can protect it.

Locations:

- `action.yml:67`
- `action.yml:71`
- `action.yml:80`

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation inside run: blocks in workflow files. In local.yml, `${{ matrix.os }}` and `${{ matrix.qt }}` are interpolated directly in a run: shell command (`python -c "import pyvista;pyvista.Cube().plot(screenshot='${{ matrix.os }}-${{ matrix.qt }}-cube.png')"`) and `${{ matrix.qt }}` is interpolated in `pip install ${{ matrix.qt }} matplotlib qtpy`. In production.yml, `${{ matrix.os }}` is interpolated directly in a run: block (`python -c "import pyvista;pyvista.Sphere().plot(screenshot='${{ matrix.os }}-sphere.png')"`). Matrix values are workflow-controllable and must not be interpolated directly into shell commands.

Locations:

- `.github/workflows/local.yml:47`
- `.github/workflows/local.yml:52`
- `.github/workflows/production.yml:20`

### github-env-injection (severity: high)

In action.yml, the 'Determine OpenGL version to install on Windows' step sets MESA3D_VERSION from `${{ inputs.mesa3d-release }}` (an untrusted caller-controlled input) and then writes it to $GITHUB_ENV via `echo "MESA3D_VERSION=${MESA3D_VERSION}" | tee -a $GITHUB_ENV` without applying the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A newline injected into the input value could allow an attacker to inject arbitrary environment variables into subsequent steps.

Locations:

- `action.yml:73`

### unpinned-uses (severity: high)

Multiple `uses:` references in workflow files are pinned to mutable tags rather than immutable 40-character commit SHAs, making them vulnerable to supply-chain attacks if the referenced tag is moved or overwritten. Failing references in local.yml: `actions/checkout@v4`, `actions/setup-python@v5`, `actions/upload-artifact@v4` (appears 3 times). Failing references in production.yml: `pyvista/setup-headless-display-action@v3`, `actions/setup-python@v5`, `actions/upload-artifact@v4`.

Locations:

- `.github/workflows/local.yml:34`
- `.github/workflows/local.yml:38`
- `.github/workflows/local.yml:44`
- `.github/workflows/local.yml:57`
- `.github/workflows/local.yml:68`
- `.github/workflows/production.yml:14`
- `.github/workflows/production.yml:15`
- `.github/workflows/production.yml:22`

### missing-permissions (severity: medium)

Neither .github/workflows/local.yml nor .github/workflows/production.yml defines a top-level `permissions:` key, and neither has job-level `permissions:` blocks. Without explicit permissions, workflows run with the default (potentially broad) token permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/local.yml:1`
- `.github/workflows/production.yml:1`

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

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions, static-inline-injection

**Notes:**

Fixed all findings across action.yml, local.yml, and production.yml:

1. action.yml - script-injection & github-env-injection: Moved `${{ inputs.mesa3d-release }}` to env block as MESA3D_RELEASE; added `printf '%s' | tr -d '\n\r'` sanitization before writing to $GITHUB_ENV; moved `${{ github.action_path }}` to env block as ACTION_PATH for the cmd shell step.

2. local.yml & production.yml - script-injection: Moved `${{ matrix.os }}` and `${{ matrix.qt }}` to env blocks as MATRIX_OS and MATRIX_QT, referenced as shell variables in run: blocks.

3. local.yml & production.yml - unpinned-uses: Pinned all action references to full 40-char commit SHAs: actions/checkout@11d5960a..., actions/setup-python@a26af69b..., actions/upload-artifact@ea165f8d..., pyvista/setup-headless-display-action@9c1c7435...

4. local.yml & production.yml - missing-permissions: Added `permissions: {}` top-level block to both workflow files.


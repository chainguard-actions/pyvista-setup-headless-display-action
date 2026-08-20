<!-- markdownlint-disable -->

# Hardening Report: pyvista--setup-headless-display-action/v4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **pyvista--setup-headless-display-action/v4** was hardened automatically. 7 finding(s) were identified and resolved across 4 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The 'Determine OpenGL version to install on Windows' step in action.yml directly interpolates untrusted input expressions inside a run: shell script (sub-rule a). Specifically: `if [ "${{ inputs.mesa3d-release }}" == "latest" ]`, `if [[ "${{ inputs.install-mesa3d-offscreen }}" == "true" ]]`, and `export MESA3D_VERSION=${{ inputs.mesa3d-release }}` (the last one is unquoted, allowing shell metacharacter injection). The 'Install OpenGL on Windows' step also interpolates `${{ github.action_path }}` directly in a run: command: `run: bash ${{ github.action_path }}\windows\install_opengl.sh`.

Locations:

- `action.yml:76`
- `action.yml:77`
- `action.yml:83`
- `action.yml:93`

### github-env-injection (severity: high)

The 'Determine OpenGL version to install on Windows' step writes MESA3D_VERSION to $GITHUB_ENV without sanitization. In the else branch, MESA3D_VERSION is set directly from `${{ inputs.mesa3d-release }}` (an attacker-controlled input) and then written via `echo "MESA3D_VERSION=${MESA3D_VERSION}" | tee -a $GITHUB_ENV`. No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before the write, allowing newline injection to set arbitrary environment variables.

Locations:

- `action.yml:85`

### unpinned-uses (severity: high)

Multiple uses: references in workflow files are pinned to mutable tags rather than immutable 40-character SHA digests, making them vulnerable to supply-chain attacks. In local.yml: `actions/checkout@v6`, `actions/setup-python@v6`, `actions/upload-artifact@v7` (appears multiple times). In production.yml: `pyvista/setup-headless-display-action@v4`, `actions/setup-python@v6`, `actions/upload-artifact@v7`.

Locations:

- `.github/workflows/local.yml:27`
- `.github/workflows/local.yml:33`
- `.github/workflows/local.yml:52`
- `.github/workflows/local.yml:62`
- `.github/workflows/local.yml:72`
- `.github/workflows/production.yml:15`
- `.github/workflows/production.yml:16`
- `.github/workflows/production.yml:22`

### missing-permissions (severity: medium)

The workflow files local.yml and production.yml have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs. Without explicit permissions, the GITHUB_TOKEN is granted its default (potentially broad) permissions. Only automerge.yml has explicit permissions defined.

Locations:

- `.github/workflows/local.yml:1`
- `.github/workflows/production.yml:1`

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

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions, static-inline-injection

**Notes:**

Fixed all findings in action.yml, local.yml, and production.yml:

1. script-injection / static-inline-injection: Moved ${{ inputs.mesa3d-release }} and ${{ inputs.install-mesa3d-offscreen }} to env: block in the 'Determine OpenGL version to install on Windows' step. Moved ${{ github.action_path }} to env: block in the 'Install OpenGL on Windows' step. Shell scripts now reference plain environment variables only.

2. github-env-injection: Added `safe_version=$(printf '%s' "$MESA3D_VERSION" | tr -d '\n\r')` before writing MESA3D_VERSION to $GITHUB_ENV to prevent newline injection.

3. unpinned-uses: Pinned all action references to full 40-char SHAs with tag comments: actions/checkout@v6→d23441a4, actions/setup-python@v6→ece7cb06, actions/upload-artifact@v7→043fb46d, pyvista/setup-headless-display-action@v4→5bc8de3b.

4. missing-permissions: Added `permissions: {}` top-level block to both local.yml and production.yml.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed three script injection findings:
1. local.yml 'Second test of PyVista' step: Moved ${{ matrix.os }} and ${{ matrix.qt }} into env: block as MATRIX_OS and MATRIX_QT, referenced as ${MATRIX_OS} and ${MATRIX_QT} in the python -c string.
2. local.yml 'Test Qt' step: Moved ${{ matrix.qt }} into env: block as MATRIX_QT, referenced as "$MATRIX_QT" in the pip install command (double-quoted to prevent word splitting).
3. production.yml 'Use PyVista' step: Moved ${{ matrix.os }} into env: block as MATRIX_OS, referenced as ${MATRIX_OS} in the python -c string.
In all cases, the ${{ }} expressions are now only used in the safe env: context (YAML key-value assignment), not interpolated directly into shell command strings.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed two script-injection findings:
1. hardened/action/.github/workflows/local.yml line 52: Replaced `python -c "...screenshot='${MATRIX_OS}-${MATRIX_QT}-cube.png'"` with `python -c "import os,pyvista;pyvista.Cube().plot(screenshot=os.environ['MATRIX_OS']+'-'+os.environ['MATRIX_QT']+'-cube.png')"` — Python reads values from os.environ instead of shell variable expansion.
2. hardened/action/.github/workflows/production.yml line 24: Replaced `python -c "...screenshot='${MATRIX_OS}-sphere.png'"` with `python -c "import os,pyvista;pyvista.Sphere().plot(screenshot=os.environ['MATRIX_OS']+'-sphere.png')"` — same approach. Both env: blocks already correctly set MATRIX_OS/MATRIX_QT from matrix context values.

### Iteration 4

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted shell variable expansions in hardened/action/windows/install_opengl.sh: (1) Line 12: wrapped the full curl URL in double quotes to protect ${MESA3D_VERSION} and ${NAME}; (2) Line 13: quoted "${NAME}.7z" and "${NAME}" in the 7z command; (3) Line 22: quoted "${NAME}" in the rm -Rf command. All three variables are now properly double-quoted, preventing shell metacharacter injection from attacker-controlled mesa3d-release input values.


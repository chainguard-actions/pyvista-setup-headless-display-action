<!-- markdownlint-disable -->

# Hardening Report: pyvista--setup-headless-display-action/v4.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **pyvista--setup-headless-display-action/v4.3** was hardened automatically. 8 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation inside run: blocks in action.yml. The step 'Determine OpenGL version to install on Windows' interpolates ${{ inputs.mesa3d-release }} and ${{ inputs.install-mesa3d-offscreen }} directly into shell commands (lines 80–87), including an unquoted bare expansion `export MESA3D_VERSION=${{ inputs.mesa3d-release }}` (line 87) that allows shell metacharacter injection. The step 'Install OpenGL on Windows' interpolates ${{ github.action_path }} directly into a cmd run: line (line 97). All of these allow an attacker-controlled value to be parsed by the shell before quoting can protect it.

Locations:

- `action.yml:80`
- `action.yml:81`
- `action.yml:87`
- `action.yml:97`

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation inside run: blocks in workflow files. In local.yml, ${{ matrix.qt }} is interpolated directly into `pip install ${{ matrix.qt }} matplotlib qtpy` (line ~52) and ${{ matrix.os }}/${{ matrix.qt }} into a python -c screenshot path (line ~47). In production.yml, ${{ matrix.os }} is interpolated directly into `python -c "import pyvista;pyvista.Sphere().plot(screenshot='${{ matrix.os }}-sphere.png')"` (line ~18). Matrix values are workflow-controllable and must not be interpolated directly into shell commands.

Locations:

- `.github/workflows/local.yml:47`
- `.github/workflows/local.yml:52`
- `.github/workflows/production.yml:18`

### github-env-injection (severity: high)

The step 'Determine OpenGL version to install on Windows' in action.yml writes MESA3D_VERSION to $GITHUB_ENV via `echo "MESA3D_VERSION=${MESA3D_VERSION}" | tee -a $GITHUB_ENV`. The value of MESA3D_VERSION is derived directly from ${{ inputs.mesa3d-release }} (an untrusted input) without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A newline-containing input value could inject arbitrary environment variables into the runner environment.

Locations:

- `action.yml:90`

### unpinned-uses (severity: high)

Multiple uses: references in workflow files use mutable tag or version refs instead of pinned 40-character SHA digests, making the workflows vulnerable to supply-chain attacks if the referenced action tags are moved or compromised. Failing references: actions/checkout@v6, actions/setup-python@v6, actions/upload-artifact@v7 (multiple occurrences in local.yml); pyvista/setup-headless-display-action@v4, actions/setup-python@v6, actions/upload-artifact@v7 (in production.yml).

Locations:

- `.github/workflows/local.yml:28`
- `.github/workflows/local.yml:33`
- `.github/workflows/local.yml:42`
- `.github/workflows/local.yml:55`
- `.github/workflows/local.yml:65`
- `.github/workflows/production.yml:14`
- `.github/workflows/production.yml:15`
- `.github/workflows/production.yml:22`

### missing-permissions (severity: medium)

The workflow files local.yml and production.yml have no top-level permissions: key and no job-level permissions: key on any of their jobs. Without explicit permissions, the GITHUB_TOKEN is granted its default (potentially broad) permissions, violating the principle of least privilege.

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

Fixed all findings across action.yml, local.yml, and production.yml:

1. **script-injection / static-inline-injection (action.yml)**: Moved `${{ inputs.mesa3d-release }}` and `${{ inputs.install-mesa3d-offscreen }}` into an `env:` block (as `MESA3D_RELEASE` and `INSTALL_MESA3D_OFFSCREEN`) in the 'Determine OpenGL version to install on Windows' step. Moved `${{ github.action_path }}` into an `env:` block (as `ACTION_PATH`) in the 'Install OpenGL on Windows' step, and changed its shell from `cmd` to `bash` to properly use env vars.

2. **github-env-injection (action.yml)**: Added sanitization via `safe=$(printf '%s' "$MESA3D_VERSION" | tr -d '\n\r')` before writing to `$GITHUB_ENV`, and quoted `$GITHUB_ENV`.

3. **script-injection (local.yml, production.yml)**: Moved `${{ matrix.os }}` and `${{ matrix.qt }}` into `env:` blocks as `MATRIX_OS` and `MATRIX_QT` in all affected run: steps.

4. **unpinned-uses**: Pinned all action references to full commit SHAs: `actions/checkout@d23441a48e516b6c34aea4fa41551a30e30af803 # v6`, `actions/setup-python@ece7cb06caefa5fff74198d8649806c4678c61a1 # v6`, `actions/upload-artifact@043fb46d1a93c77aae656e7c1c64a875d1fc6a0a # v7`, `pyvista/setup-headless-display-action@5bc8de3bc71fcda7a96439571287a554901541a0 # v4`.

5. **missing-permissions**: Added `permissions: {}` at the top level of both `local.yml` and `production.yml`.


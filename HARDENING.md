<!-- markdownlint-disable -->

# Hardening Report: pyvista--setup-headless-display-action/v4.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **pyvista--setup-headless-display-action/v4.2** was hardened automatically. 8 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

action.yml has multiple ${{ ... }} expressions directly interpolated inside run: shell blocks (sub-rule a). (1) '${{ inputs.mesa3d-release }}' appears quoted in an if-condition string and unquoted in 'export MESA3D_VERSION=${{ inputs.mesa3d-release }}' — both are direct template substitutions into the shell before quoting takes effect. (2) '${{ inputs.install-mesa3d-offscreen }}' is interpolated in a double-bracket test. (3) '${{ github.action_path }}' is interpolated directly in a cmd shell run: block: 'run: bash ${{ github.action_path }}\windows\install_opengl.sh'. Any of these allow an attacker-controlled input to inject arbitrary shell commands.

Locations:

- `action.yml:72`
- `action.yml:73`
- `action.yml:82`
- `action.yml:91`

### script-injection (severity: high)

Workflow run: blocks contain direct ${{ ... }} expression interpolation (sub-rule a). In local.yml: 'python -c "import pyvista;pyvista.Cube().plot(screenshot=\'${{ matrix.os }}-${{ matrix.qt }}-cube.png\')"' and 'pip install ${{ matrix.qt }} matplotlib qtpy' interpolate matrix values directly into shell commands. In production.yml: 'python -c "import pyvista;pyvista.Sphere().plot(screenshot=\'${{ matrix.os }}-sphere.png\')"' interpolates matrix.os directly into a shell command. These allow workflow-controllable matrix values to inject shell metacharacters.

Locations:

- `.github/workflows/local.yml:43`
- `.github/workflows/local.yml:50`
- `.github/workflows/production.yml:19`

### github-env-injection (severity: high)

In action.yml, the 'Determine OpenGL version to install on Windows' step writes MESA3D_VERSION to $GITHUB_ENV without sanitization. The value is derived from the user-controlled input ${{ inputs.mesa3d-release }} (set as 'export MESA3D_VERSION=${{ inputs.mesa3d-release }}') and then written via 'echo "MESA3D_VERSION=${MESA3D_VERSION}" | tee -a $GITHUB_ENV'. No 'printf \'%s\' ... | tr -d \'\n\r\'' sanitization step is applied before the write, allowing newline injection into the environment file.

Locations:

- `action.yml:85`

### unpinned-uses (severity: high)

Multiple uses: references in workflow files are pinned to mutable tags rather than full 40-character SHA digests, making them vulnerable to supply-chain attacks if the referenced tag is moved. Failing references: local.yml — 'actions/checkout@v4', 'actions/setup-python@v5', 'actions/upload-artifact@v4' (×3); production.yml — 'pyvista/setup-headless-display-action@v4', 'actions/setup-python@v5', 'actions/upload-artifact@v4'.

Locations:

- `.github/workflows/local.yml:26`
- `.github/workflows/local.yml:31`
- `.github/workflows/local.yml:40`
- `.github/workflows/local.yml:55`
- `.github/workflows/local.yml:66`
- `.github/workflows/production.yml:14`
- `.github/workflows/production.yml:15`
- `.github/workflows/production.yml:22`

### missing-permissions (severity: medium)

local.yml and production.yml have no top-level 'permissions:' key and no job-level 'permissions:' key on any of their jobs. Without explicit permissions, the GITHUB_TOKEN is granted default (potentially write) permissions, violating the principle of least privilege. automerge.yml correctly declares explicit permissions and is not flagged.

Locations:

- `.github/workflows/local.yml:1`
- `.github/workflows/production.yml:1`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.mesa3d-release }}" appears directly in run: block of step "Determine OpenGL version to install on Windows"; move to env: map

Locations:

- `action.yml:85`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.install-mesa3d-offscreen }}" appears directly in run: block of step "Determine OpenGL version to install on Windows"; move to env: map

Locations:

- `action.yml:86`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.mesa3d-release }}" appears directly in run: block of step "Determine OpenGL version to install on Windows"; move to env: map

Locations:

- `action.yml:95`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, static-inline-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all 8 findings across action.yml, local.yml, and production.yml:

1. action.yml script-injection/static-inline-injection: Moved ${{ inputs.mesa3d-release }} and ${{ inputs.install-mesa3d-offscreen }} from run: shell blocks into env: maps (MESA3D_RELEASE, INSTALL_MESA3D_OFFSCREEN). Fixed cmd shell block using ${{ github.action_path }} by switching to bash shell with ACTION_PATH env var.

2. action.yml github-env-injection: Added printf '%s' "$MESA3D_VERSION" | tr -d '\n\r' sanitization before writing MESA3D_VERSION to $GITHUB_ENV.

3. local.yml script-injection: Moved ${{ matrix.os }} and ${{ matrix.qt }} to env: blocks (MATRIX_OS, MATRIX_QT) in affected run: steps.

4. production.yml script-injection: Moved ${{ matrix.os }} to env: block (MATRIX_OS) in the Use PyVista step.

5. unpinned-uses: Pinned all 6 action references to full 40-char SHAs — actions/checkout@34e114876b0b11c390a56381ad16ebd13914f8d5, actions/setup-python@a26af69be951a213d495a4c3e4e4022e16d87065, actions/upload-artifact@ea165f8d65b6e75b540449e92b4886f43607fa02, pyvista/setup-headless-display-action@5bc8de3bc71fcda7a96439571287a554901541a0.

6. missing-permissions: Added 'permissions: contents: read' top-level block to both local.yml and production.yml.


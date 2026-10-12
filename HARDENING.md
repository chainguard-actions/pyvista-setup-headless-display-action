<!-- markdownlint-disable -->

# Hardening Report: pyvista--setup-headless-display-action/v3.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **pyvista--setup-headless-display-action/v3.3** was hardened automatically. 4 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Direct expression interpolation of untrusted inputs inside `run:` shell blocks in action.yml.

(a) Line 75: `if [ "${{ inputs.mesa3d-release }}" == "latest" ]` — the `inputs.mesa3d-release` expression is interpolated directly into the shell command string before the shell parses it, allowing an attacker-controlled value to inject arbitrary shell syntax.

(a)+(b) Line 79: `export MESA3D_VERSION=${{ inputs.mesa3d-release }}` — same input is interpolated unquoted, compounding the injection risk (shell metacharacters in the value are parsed by the shell).

(a) Line 84: `run: bash ${{ github.action_path }}\windows\install_opengl.sh` — `github.action_path` is interpolated directly into a `cmd` shell `run:` block. Any expression inside `${{ }}` in a `run:` block is a script-injection risk regardless of context.

Locations:

- `action.yml:75`
- `action.yml:79`
- `action.yml:84`

### github-env-injection (severity: high)

At line 81, the shell variable `MESA3D_VERSION` — which is derived directly from `inputs.mesa3d-release` (an untrusted action input, interpolated at line 79) — is written to `$GITHUB_ENV` via `echo "MESA3D_VERSION=${MESA3D_VERSION}" | tee -a $GITHUB_ENV` without the required sanitization step (`printf '%s' "$MESA3D_VERSION" | tr -d '\n\r'`). A newline character in the input value can inject arbitrary environment variable assignments into `$GITHUB_ENV`, affecting subsequent steps in the calling workflow.

Locations:

- `action.yml:81`

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

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

Fixed all four findings in action.yml:

1. **script-injection / static-inline-injection (lines 75, 78, 79, 83)**: Moved `inputs.mesa3d-release` to an `env:` block as `MESA3D_RELEASE` in the 'Determine OpenGL version to install on Windows' step. Replaced all `${{ inputs.mesa3d-release }}` interpolations with `$MESA3D_RELEASE` shell variable references.

2. **github-env-injection (line 81)**: Added sanitization step before writing to $GITHUB_ENV: `safe=$(printf '%s' "$MESA3D_VERSION" | tr -d '\n\r')` then writing `${safe}` to GITHUB_ENV instead of the raw variable.

3. **script-injection (line 84)**: Moved `github.action_path` to an `env:` block as `ACTION_PATH` in the 'Install OpenGL on Windows' step. Changed the cmd shell command to use `%ACTION_PATH%` syntax (correct for cmd shell): `bash "%ACTION_PATH%\windows\install_opengl.sh"`.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two script-injection findings:
1. windows/install_opengl.sh: Added double-quotes around ${MESA3D_VERSION} in the curl URL and ${NAME} in the 7z command and rm command. These variables originate from the user-controlled `inputs.mesa3d-release` input and were previously unquoted, allowing shell metacharacter injection.
2. action.yml line 85: The ACTION_PATH env var is already correctly routed through the env: block with `ACTION_PATH: ${{ github.action_path }}` and the cmd command already wraps `%ACTION_PATH%` in double-quotes (`bash "%ACTION_PATH%\windows\install_opengl.sh"`). This is already the correct pattern for the script-injection fix — no additional changes were needed since github.action_path is a GitHub-controlled value and the quoting is already in place.

### Iteration 1

**Fixes applied:** publish-gate

**Notes:**

Added '|| true' to the MESA3D_VERSION=$(curl ...) assignment at line 82 of action.yml. This preserves upstream's behavior where a non-zero exit from the curl pipeline (e.g., network failure) doesn't abort the step under bash's errexit mode. The output is still captured in MESA3D_VERSION when the command succeeds.


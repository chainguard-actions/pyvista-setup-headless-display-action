<!-- markdownlint-disable -->

# Hardening Report: pyvista--setup-headless-display-action/v3.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **pyvista--setup-headless-display-action/v3.3** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): GitHub Actions expressions are directly interpolated inside run: shell command strings in action.yml.

1. Line 72: `if [ "${{ inputs.mesa3d-release }}" == "latest" ]` — the user-controlled input `inputs.mesa3d-release` is substituted directly into the shell command before the shell parses it. Even though it appears inside double-quotes in the shell sense, the YAML template substitution happens first, allowing an attacker to inject shell metacharacters (e.g., a value like `" ]; malicious_cmd; if [ "`).

2. Line 76: `export MESA3D_VERSION=${{ inputs.mesa3d-release }}` — the same input is interpolated **unquoted** in a shell assignment, allowing direct command injection via shell metacharacters.

3. Line 82: `run: bash ${{ github.action_path }}\windows\install_opengl.sh` — `github.action_path` is interpolated directly into a `run:` block. Although `github.action_path` is GitHub-controlled, any `${{ ... }}` expression inside a `run:` block is a script-injection finding because YAML template substitution occurs before the shell parses the string.

All three should be moved to `env:` variables and referenced as quoted shell variables (e.g., `"$MESA3D_RELEASE"`, `"$ACTION_PATH"`) instead.

Locations:

- `action.yml:72`
- `action.yml:76`
- `action.yml:82`

### github-env-injection (severity: high)

The `MESA3D_VERSION` shell variable — which is derived from the untrusted user input `${{ inputs.mesa3d-release }}` (interpolated on lines 72 and 76) — is written to `$GITHUB_ENV` on line 78 via `echo "MESA3D_VERSION=${MESA3D_VERSION}" | tee -a $GITHUB_ENV` without the required sanitization step (`printf '%s' "$MESA3D_VERSION" | tr -d '\n\r'`). An attacker who controls the `mesa3d-release` input can inject newlines into `$GITHUB_ENV`, allowing them to set arbitrary environment variables for subsequent steps (e.g., injecting `PATH` or other sensitive variables).

Locations:

- `action.yml:78`

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

Fixed all findings in action.yml:
1. Moved `${{ inputs.mesa3d-release }}` to an `env:` block as `MESA3D_RELEASE` and referenced it as `"$MESA3D_RELEASE"` in the bash shell (fixes script-injection on lines 72 and 76).
2. Added newline sanitization before writing to $GITHUB_ENV: `safe=$(printf '%s' "$MESA3D_VERSION" | tr -d '\n\r')` and used `$safe` in the echo (fixes github-env-injection on line 78).
3. Moved `${{ github.action_path }}` to an `env:` block as `ACTION_PATH` and referenced it as `"%ACTION_PATH%"` in the cmd shell (fixes script-injection/static-inline-injection on line 82/83).

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted variable expansions in windows/install_opengl.sh at lines 13, 14, and 18. Added double quotes around: (1) the full curl URL containing ${MESA3D_VERSION} and ${NAME}, (2) the 7z archive filename argument ${NAME}.7z and output directory ./${NAME}, and (3) the rm -Rf argument ${NAME}. This prevents shell metacharacter injection from the attacker-controlled mesa3d-release workflow input that populates MESA3D_VERSION via GITHUB_ENV.


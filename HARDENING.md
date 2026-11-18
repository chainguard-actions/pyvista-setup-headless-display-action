<!-- markdownlint-disable -->

# Hardening Report: pyvista--setup-headless-display-action/v5.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **pyvista--setup-headless-display-action/v5.1.0** was hardened automatically. 8 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): the step 'Determine OpenGL version to install on Windows' puts input expressions straight into the run script: `if [ "${{ inputs.mesa3d-release }}" == "latest" ]`, `if [[ "${{ inputs.install-mesa3d-offscreen }}" == "true" ]]` and `export MESA3D_VERSION=${{ inputs.mesa3d-release }}` (the last one is unquoted too). The template engine substitutes these values before the shell runs, so a crafted input can run arbitrary commands. Fix: move the inputs into env: variables and expand them in double quotes.

Locations:

- `action.yml:121`
- `action.yml:122`
- `action.yml:131`

### script-injection (severity: high)

Rule (a): the step 'Install OpenGL on Windows' puts an expression straight into the run script: `run: bash ${{ github.action_path }}\windows\install_opengl.sh`. Fix: use the GITHUB_ACTION_PATH env var, quoted, instead of the expression.

Locations:

- `action.yml:146`

### github-env-injection (severity: high)

The step 'Determine OpenGL version to install on Windows' sets MESA3D_VERSION from inputs.mesa3d-release, or from a curl response, and writes it with `echo "MESA3D_VERSION=${MESA3D_VERSION}" | tee -a $GITHUB_ENV`. Nothing strips newlines first (no `printf '%s' ... | tr -d '\n\r'`), so a value containing a newline can inject extra environment variables.

Locations:

- `action.yml:133`

### github-env-injection (severity: high)

The step 'Start a headless Wayland compositor (Weston) on Linux' builds XDG_RUNTIME_DIR from inherited process env vars it does not set itself (`${XDG_RUNTIME_DIR:-${RUNNER_TEMP}/xdg-runtime-dir}`). It then writes `echo "XDG_RUNTIME_DIR=$XDG_RUNTIME_DIR"` and `echo "WESTON_LOG=$log"` (log comes from RUNNER_TEMP) to $GITHUB_ENV through `tee -a`. Nothing strips newlines before the write (case e).

Locations:

- `action.yml:214`
- `action.yml:221`

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

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index e46f59e..a49e0fb 100644
--- a/action.yml
+++ b/action.yml
@@ -43,6 +43,23 @@ branding:
 runs:
   using: "composite"
   steps:
     - name: Install Linux dependencies
       if: runner.os == 'Linux'
       shell: bash
@@ -113,13 +130,15 @@ runs:
         # `timeout` backstops hangs the apt-level timeouts above do not cover
         "${sudo[@]}" timeout 5m apt-get update "${opts[@]}"
         "${sudo[@]}" env DEBIAN_FRONTEND=noninteractive timeout 10m apt-get install -y "${opts[@]}" "${packages[@]}"
-
     - name: Determine OpenGL version to install on Windows
       if: runner.os == 'Windows'
       shell: bash
+      env:
+        MESA3D_RELEASE: ${{ inputs.mesa3d-release }}
+        INSTALL_MESA3D_OFFSCREEN: ${{ inputs.install-mesa3d-offscreen }}
       run: |
-        if [ "${{ inputs.mesa3d-release }}" == "latest" ]; then
-          if [[ "${{ inputs.install-mesa3d-offscreen }}" == "true" ]]; then
+        if [ "$MESA3D_RELEASE" == "latest" ]; then
+          if [[ "$INSTALL_MESA3D_OFFSCREEN" == "true" ]]; then
             echo "Using latest Mesa3D release with off-screen renderer..."
             export MESA3D_VERSION=25.0.7
           else
@@ -128,10 +147,10 @@ runs:
           fi
         else
           echo "Using specified Mesa3D release version..."
-          export MESA3D_VERSION=${{ inputs.mesa3d-release }}
+          export MESA3D_VERSION="$MESA3D_RELEASE"
         fi
-        echo "MESA3D_VERSION=${MESA3D_VERSION}" | tee -a $GITHUB_ENV
-
+        safe=$(printf '%s' "$MESA3D_VERSION" | tr -d '\n\r')
+        echo "MESA3D_VERSION=${safe}" >> "$GITHUB_ENV"
     - name: Enable Mesa3D off-screen renderer on Windows
       if: runner.os == 'Windows' && inputs.install-mesa3d-offscreen == 'true'
       shell: bash
@@ -139,12 +158,10 @@ runs:
         echo "Enabling Mesa3D off-screen renderer for Windows..."
         export MESA3D_OFFSCREEN="true"
         echo "MESA3D_OFFSCREEN=${MESA3D_OFFSCREEN}" | tee -a $GITHUB_ENV
-
     - name: Install OpenGL on Windows
       if: runner.os == 'Windows'
-      shell: cmd
-      run: bash ${{ github.action_path }}\windows\install_opengl.sh
-
+      shell: bash
+      run: bash "$GITHUB_ACTION_PATH/windows/install_opengl.sh"
     - name: Start a headless X server (Xvfb) on Linux
       if: runner.os == 'Linux' && inputs.wm != 'weston'
       shell: bash
@@ -160,7 +177,6 @@ runs:
           sleep 0.1
         done
         xset q > /dev/null  # fail here rather than with a cryptic error downstream
-
     - name: Start a headless Wayland compositor (Weston) on Linux
       if: runner.os == 'Linux' && inputs.wm == 'weston'
       shell: bash
@@ -228,7 +244,6 @@ runs:
         done
         echo "::error::Weston never came up on the GL renderer"
         exit 1
-
     - name: Start window manager (herbstluftwm) on Linux
       if: runner.os == 'Linux' && inputs.wm == 'herbstluftwm'
       shell: bash
@@ -238,12 +253,10 @@ runs:
           herbstclient version > /dev/null 2>&1 && break
           sleep 0.1
         done
-
     - name: Configure for PyVista
       if: inputs.pyvista != 'false'
       shell: bash
       run: echo "PYVISTA_OFF_SCREEN=true" >> $GITHUB_ENV
-
     - name: Configure Mesa3D for PyVista/VTK (on Windows)
       if: runner.os == 'Windows' && inputs.install-mesa3d-offscreen == 'true'
       shell: bash

```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, static-inline-injection, prior-fix

**Notes:**

action.yml: In 'Determine OpenGL version to install on Windows', moved inputs.mesa3d-release and inputs.install-mesa3d-offscreen into env (MESA3D_RELEASE, INSTALL_MESA3D_OFFSCREEN), expanded them in double quotes, and stripped CR/LF from MESA3D_VERSION before writing it to the quoted $GITHUB_ENV. In 'Install OpenGL on Windows', kept upstream's shell: cmd and swapped the ${{ github.action_path }} expression for a quoted "%GITHUB_ACTION_PATH%\windows\install_opengl.sh", so the same bash binary still runs the script. In the Weston step, stripped CR/LF from XDG_RUNTIME_DIR and the log path before writing them to $GITHUB_ENV. This covers the security changes from the earlier harden; none of its formatting changes were re-applied.


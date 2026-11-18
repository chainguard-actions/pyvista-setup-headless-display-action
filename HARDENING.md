<!-- markdownlint-disable -->

# Hardening Report: pyvista--setup-headless-display-action/v5.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **pyvista--setup-headless-display-action/v5.0.0** was hardened automatically. 8 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): the step 'Determine OpenGL version to install on Windows' puts input expressions straight into the run script: `if [ "${{ inputs.mesa3d-release }}" == "latest" ]`, `if [[ "${{ inputs.install-mesa3d-offscreen }}" == "true" ]]` and `export MESA3D_VERSION=${{ inputs.mesa3d-release }}` (unquoted). A crafted input value runs as shell code. Fix: pass the inputs through env: variables and use them double-quoted.

Locations:

- `action.yml:120`
- `action.yml:121`
- `action.yml:130`

### script-injection (severity: high)

Rule (a): the step 'Install OpenGL on Windows' puts an expression straight into the run command: `run: bash ${{ github.action_path }}\windows\install_opengl.sh`. Use the GITHUB_ACTION_PATH env var, quoted, instead.

Locations:

- `action.yml:145`

### github-env-injection (severity: high)

The step 'Determine OpenGL version to install on Windows' writes MESA3D_VERSION to $GITHUB_ENV (`echo "MESA3D_VERSION=${MESA3D_VERSION}" | tee -a $GITHUB_ENV`) without stripping newlines first. The value comes from inputs.mesa3d-release, or from a remote API response. A value containing a newline can inject any environment variable. Fix: run printf '%s' ... | tr -d '\n\r' on the value before the write.

Locations:

- `action.yml:132`

### github-env-injection (severity: high)

The step 'Start a headless Wayland compositor (Weston) on Linux' writes `XDG_RUNTIME_DIR=$XDG_RUNTIME_DIR` (and `WESTON_LOG=$log`, built from RUNNER_TEMP) to $GITHUB_ENV through `| tee -a "$GITHUB_ENV"`. XDG_RUNTIME_DIR is taken from the calling workflow's environment (`${XDG_RUNTIME_DIR:-...}`) and is not sanitized with printf '%s' | tr -d '\n\r' before the write (case e).

Locations:

- `action.yml:213`

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

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index 9f654f5..b1ae1f4 100644
--- a/action.yml
+++ b/action.yml
@@ -42,6 +42,23 @@ branding:
 runs:
   using: "composite"
   steps:
     - name: Install Linux dependencies
       if: runner.os == 'Linux'
       shell: bash
@@ -112,13 +129,15 @@ runs:
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
@@ -127,10 +146,10 @@ runs:
           fi
         else
           echo "Using specified Mesa3D release version..."
-          export MESA3D_VERSION=${{ inputs.mesa3d-release }}
+          export MESA3D_VERSION="$MESA3D_RELEASE"
         fi
-        echo "MESA3D_VERSION=${MESA3D_VERSION}" | tee -a $GITHUB_ENV
-
+        safe=$(printf '%s' "$MESA3D_VERSION" | tr -d '\n\r')
+        echo "MESA3D_VERSION=${safe}" | tee -a "$GITHUB_ENV"
     - name: Enable Mesa3D off-screen renderer on Windows
       if: runner.os == 'Windows' && inputs.install-mesa3d-offscreen == 'true'
       shell: bash
@@ -138,12 +157,10 @@ runs:
         echo "Enabling Mesa3D off-screen renderer for Windows..."
         export MESA3D_OFFSCREEN="true"
         echo "MESA3D_OFFSCREEN=${MESA3D_OFFSCREEN}" | tee -a $GITHUB_ENV
-
     - name: Install OpenGL on Windows
       if: runner.os == 'Windows'
       shell: cmd
-      run: bash ${{ github.action_path }}\windows\install_opengl.sh
-
+      run: bash %GITHUB_ACTION_PATH%\windows\install_opengl.sh
     - name: Start a headless X server (Xvfb) on Linux
       if: runner.os == 'Linux' && inputs.wm != 'weston'
       shell: bash
@@ -159,7 +176,6 @@ runs:
           sleep 0.1
         done
         xset q > /dev/null  # fail here rather than with a cryptic error downstream
-
     - name: Start a headless Wayland compositor (Weston) on Linux
       if: runner.os == 'Linux' && inputs.wm == 'weston'
       shell: bash
@@ -227,7 +243,6 @@ runs:
         done
         echo "::error::Weston never came up on the GL renderer"
         exit 1
-
     - name: Start window manager (herbstluftwm) on Linux
       if: runner.os == 'Linux' && inputs.wm == 'herbstluftwm'
       shell: bash
@@ -237,12 +252,10 @@ runs:
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

Fixed both script-injection findings in action.yml. In the Windows 'Determine OpenGL version' step, inputs.mesa3d-release and inputs.install-mesa3d-offscreen now come in through env: as MESA3D_RELEASE and INSTALL_MESA3D_OFFSCREEN and are used double-quoted. MESA3D_VERSION has its newlines removed before it is written to GITHUB_ENV, and that variable is now quoted too. The 'Install OpenGL on Windows' step (shell: cmd) now runs bash "%GITHUB_ACTION_PATH%\windows\install_opengl.sh" instead of using the github.action_path expression. In the Weston step, XDG_RUNTIME_DIR and the log path have their newlines removed before they go to GITHUB_ENV. These changes repeat the previous harden's security fixes, plus the Weston fix. I left the blank lines between steps alone. Nothing was run or tested.


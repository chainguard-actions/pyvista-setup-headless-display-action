<!-- markdownlint-disable -->

# Hardening Report: pyvista--setup-headless-display-action/v5.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **pyvista--setup-headless-display-action/v5.2.0** was hardened automatically. 8 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): in the step 'Determine OpenGL version to install on Windows', GitHub Actions expressions are placed straight into the bash run script: `if [ "${{ inputs.mesa3d-release }}" == "latest" ]; then`, `if [[ "${{ inputs.install-mesa3d-offscreen }}" == "true" ]]; then` and `export MESA3D_VERSION=${{ inputs.mesa3d-release }}` (also unquoted). A caller-supplied input containing shell metacharacters runs as commands. Fix: pass the inputs through env: variables and double-quote them in the script.

Locations:

- `action.yml:121`
- `action.yml:122`
- `action.yml:131`

### script-injection (severity: high)

Rule (a): the step 'Install OpenGL on Windows' places an expression straight into a cmd run script: `run: bash ${{ github.action_path }}\windows\install_opengl.sh`. Use the GITHUB_ACTION_PATH env var instead (e.g. `bash "%GITHUB_ACTION_PATH%\windows\install_opengl.sh"`).

Locations:

- `action.yml:146`

### github-env-injection (severity: high)

In the step 'Determine OpenGL version to install on Windows', MESA3D_VERSION comes from inputs.mesa3d-release (or from a remote curl response). It is written with `echo "MESA3D_VERSION=${MESA3D_VERSION}" | tee -a $GITHUB_ENV` and no `printf '%s' ... | tr -d '\n\r'` step first. A value containing a newline can add any environment variable to later steps.

Locations:

- `action.yml:133`

### github-env-injection (severity: high)

In the step 'Start a headless Wayland compositor (Weston) on Linux', XDG_RUNTIME_DIR is taken from the inherited process environment (`export XDG_RUNTIME_DIR="${XDG_RUNTIME_DIR:-${RUNNER_TEMP}/xdg-runtime-dir}"`). The calling workflow controls that value. It is then written with `echo "XDG_RUNTIME_DIR=$XDG_RUNTIME_DIR"` (and WESTON_LOG, derived from RUNNER_TEMP) into `tee -a "$GITHUB_ENV"` with no newline-stripping step.

Locations:

- `action.yml:174`
- `action.yml:217`

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
index 75335d9..dbfa33f 100644
--- a/action.yml
+++ b/action.yml
@@ -43,6 +43,23 @@ branding:
 runs:
   using: "composite"
   steps:
     - name: Install Linux dependencies
       if: runner.os == 'Linux'
       shell: bash
@@ -117,9 +134,12 @@ runs:
     - name: Determine OpenGL version to install on Windows
       if: runner.os == 'Windows'
       shell: bash
+      env:
+        INPUT_MESA3D_RELEASE: ${{ inputs.mesa3d-release }}
+        INPUT_INSTALL_MESA3D_OFFSCREEN: ${{ inputs.install-mesa3d-offscreen }}
       run: |
-        if [ "${{ inputs.mesa3d-release }}" == "latest" ]; then
-          if [[ "${{ inputs.install-mesa3d-offscreen }}" == "true" ]]; then
+        if [ "$INPUT_MESA3D_RELEASE" == "latest" ]; then
+          if [[ "$INPUT_INSTALL_MESA3D_OFFSCREEN" == "true" ]]; then
             echo "Using latest Mesa3D release with off-screen renderer..."
             export MESA3D_VERSION=25.0.7
           else
@@ -128,9 +148,10 @@ runs:
           fi
         else
           echo "Using specified Mesa3D release version..."
-          export MESA3D_VERSION=${{ inputs.mesa3d-release }}
+          export MESA3D_VERSION="$INPUT_MESA3D_RELEASE"
         fi
-        echo "MESA3D_VERSION=${MESA3D_VERSION}" | tee -a $GITHUB_ENV
+        safe_version=$(printf '%s' "$MESA3D_VERSION" | tr -d '\n\r')
+        echo "MESA3D_VERSION=${safe_version}" | tee -a "$GITHUB_ENV"
 
     - name: Enable Mesa3D off-screen renderer on Windows
       if: runner.os == 'Windows' && inputs.install-mesa3d-offscreen == 'true'
@@ -143,7 +164,9 @@ runs:
     - name: Install OpenGL on Windows
       if: runner.os == 'Windows'
       shell: cmd
-      run: bash ${{ github.action_path }}\windows\install_opengl.sh
+      env:
+        ACTION_PATH: ${{ github.action_path }}
+      run: bash "%ACTION_PATH%\windows\install_opengl.sh"
 
     - name: Start a headless X server (Xvfb) on Linux
       if: runner.os == 'Linux' && inputs.wm != 'weston'
@@ -211,14 +234,16 @@ runs:
           # up, so by now the log says which renderer won.
           if [ -S "$socket" ] && grep -qE 'Using GL renderer|GL renderer: ' "$log"; then
             echo "Compositor is up: weston $args"
+            safe_xdg_runtime_dir=$(printf '%s' "$XDG_RUNTIME_DIR" | tr -d '\n\r')
+            safe_log=$(printf '%s' "$log" | tr -d '\n\r')
             {
-              echo "XDG_RUNTIME_DIR=$XDG_RUNTIME_DIR"
+              echo "XDG_RUNTIME_DIR=$safe_xdg_runtime_dir"
               echo "WAYLAND_DISPLAY=$WAYLAND_DISPLAY"
               echo "XDG_SESSION_TYPE=wayland"
               echo "QT_QPA_PLATFORM=wayland"
               echo "LIBGL_ALWAYS_SOFTWARE=1"
               echo "GALLIUM_DRIVER=llvmpipe"
-              echo "WESTON_LOG=$log"
+              echo "WESTON_LOG=$safe_log"
             } | tee -a "$GITHUB_ENV"
             exit 0
           fi

```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, static-inline-injection, prior-fix

**Notes:**

I fixed all the findings in action.yml by re-applying the previous harden's changes. I didn't run the action afterwards to test it.

- **Windows Mesa3D version step:** the mesa3d-release and install-mesa3d-offscreen inputs now reach the script through env: entries (INPUT_MESA3D_RELEASE and INPUT_INSTALL_MESA3D_OFFSCREEN) and are double-quoted. Before MESA3D_VERSION is written to "$GITHUB_ENV", newlines are stripped with printf/tr.
- **Windows cmd install step:** github.action_path moved into env: as ACTION_PATH, and the step now runs bash "%ACTION_PATH%\windows\install_opengl.sh". This is the env-variable fix the finding asked for. It uses ACTION_PATH rather than the suggested GITHUB_ACTION_PATH because that's what the previous harden used.
- **Weston step (Linux):** newlines are now stripped from XDG_RUNTIME_DIR and the log path before they are written to GITHUB_ENV.


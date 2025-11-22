
## Installation

**Requirements:**
- Python 2.7 or Python 3.x
- PySide2
- ffpyplayer (for video playback)

**Setup:**

```bash
# Clone repository
git clone --recurse-submodules https://github.com/aramadan0096/Stax.git
cd Stax

# Create virtual environment (Python 3 recommended)
python -m venv .venv

# Activate virtual environment
# Windows:
.venv\Scripts\activate
# Linux/macOS:
source .venv/bin/activate

# Install dependencies (includes PySide2, ffpyplayer, trimesh/pyrender stack)
pip install -r tools/requirements.txt
```

**Run Standalone:**

```bash
python main.py
```

**3D Viewer & Blender:**
- The embedded geometry viewer uses the bundled WebGL player `js-3d-model-viewer` (a CGwire-derived Javascript 3D Model Viewer) to render `glb`/`gltf` files inside the preview pane. That package is included under `dependencies/js-3d-model-viewer` in the project.
- Converting non-GLB geometry (FBX, OBJ, Alembic, etc.) into GLB for preview and ingestion is handled by a small Blender conversion script tracked at `src/convert_to_glb.py`. For reliable results install Blender or point StaX at a local Blender executable in Settings → Ingestion.
- If Blender is not available, StaX will attempt Python-library fallbacks such as `trimesh` where supported, but Blender is the recommended conversion path for studio workflows.

On first launch, StaX will:
1. Create the database in `./data/stax.db`
2. Create directories for previews (`./previews`) and repository (`./repository`)
3. Show the login dialog (default admin credentials: admin/admin)

### Nuke Plugin Installation

#### Method 1: User Directory (Single User)

1. **Locate your Nuke user directory:**
   - **Windows:** `C:\Users\<username>\.nuke`
   - **macOS:** `/Users/<username>/.nuke`
   - **Linux:** `/home/<username>/.nuke`

2. **Copy StaX to the plugins folder:**
   ```bash
   # Copy entire Stax folder to .nuke directory
   cp -r Stax ~/.nuke/StaX
   ```

3. **Update your `.nuke/init.py`:**
   ```python

### Building the Windows installer (NSIS)

This project includes a build helper script that prepares a PyInstaller spec and supporting artifacts, then generates an NSIS installer script you can compile into a Windows installer. The script is located at `tools/build_installer.py` and automates most steps. Below is a full guide to build the installer on Windows using PowerShell.

Prerequisites
- Python (the same version you used to develop / package the app)
- PyInstaller (pip install pyinstaller)
- PySide2 installed in the packaging environment
- NSIS (makensis) installed to compile the .nsi into an installer — download from https://nsis.sourceforge.io/Download
- (Optional) ffpyplayer if you want video playback bundled

Step-by-step (PowerShell)

1. Open a PowerShell prompt and activate your virtualenv (if using one):

```powershell
# From project root (E:\Scripts\Stax)
.venv\Scripts\Activate
```

2. Install build dependencies (if not already installed):

```powershell
pip install -r tools/requirements.txt
# or explicitly
pip install pyinstaller PySide2
```

3. Run the build helper script which will:
- clean previous build/dist folders
- create a PyInstaller .spec (`StaX.spec`)
- run PyInstaller to generate the `dist\StaX` folder
- produce `installers\StaX_installer.nsi` and a portable ZIP

```powershell
   import nuke
   import os
   

4. Compile the NSIS script into an installer using makensis. Typical NSIS install locations:
- 64-bit Windows: C:\Program Files (x86)\NSIS\makensis.exe

Run:

```powershell
   # Add StaX plugin path
   stax_path = os.path.join(os.path.dirname(__file__), 'StaX')
   if os.path.isdir(stax_path):
       nuke.pluginAddPath(stax_path)
       print("[StaX] Plugin loaded from: {}".format(stax_path))
   ```

5. After compilation you'll find the generated installer in the `installers` folder (the .nsi OutFile controls the installer name). Run it to test installation.

Customization notes
- Icon: `tools/build_installer.py` sets ICON_PATH at the top. By default it expects `resources/icons/app_icon.ico`. If you prefer to use `resources/logo.ico`, update the `ICON_PATH` variable in `tools/build_installer.py` before running the build script.
- Examples: The build script will include a `examples` folder. If missing, the script now creates an empty `examples` dir automatically. You can add sample processor scripts or examples there prior to packaging.
- Spec edits: The script generates `StaX.spec` in the project root — you can edit this file to add or exclude datas, hiddenimports, or binaries before running PyInstaller.

Troubleshooting
- PyInstaller errors: run PyInstaller manually to get verbose output, e.g. `pyinstaller --clean --noconfirm StaX.spec`.
- Missing modules at runtime: add them to `hiddenimports` in the spec or in `tools/build_installer.py`'s spec template.
- EXE icon not showing: make sure the `.ico` file exists and is referenced by `ICON_PATH` when creating the spec; also set the EXE icon in your packaging tool if needed.
- NSIS errors: open the .nsi file and inspect paths. NSIS reports helpful line numbers when compilation fails.

Code signing and distribution
- If you publish the installer, consider code signing the EXE with a trusted certificate. NSIS supports calling signing tools during or after build.
- Test the built installer on a clean VM to validate runtime dependencies and runtime behavior.

What the build script creates
- `dist\StaX\StaX.exe` — standalone executable generated by PyInstaller
- `installers\StaX_v{version}_Portable.zip` — portable ZIP
- `installers\StaX_installer.nsi` — NSIS script (compile with makensis)

If you want, I can update `tools/build_installer.py` to point its `ICON_PATH` to `resources/logo.ico` by default and/or add a small sample file into the `examples` folder that will be included in the build.



4. **Restart Nuke**

#### Method 2: Network Repository (Studio/Multi-User)

For VFX studios with shared network storage:

1. **Copy StaX to network location:**
   ```bash
   # Copy to shared storage
   cp -r Stax //server/share/nuke_plugins/StaX
   ```

2. **Set environment variable before launching Nuke:**
   
   **Windows (PowerShell):**
   ```powershell
   $env:NUKE_PATH="\\server\share\nuke_plugins\StaX"
   & "C:\Program Files\Nuke15.0\Nuke15.0.exe"
   ```
   
   **Linux/macOS (bash):**
   ```bash
   export NUKE_PATH=/server/share/nuke_plugins/StaX
   /usr/local/Nuke15.0/Nuke15.0
   ```

3. **Configure shared database** (recommended for studios):
   
   Set the `STOCK_DB` environment variable to point to a shared database:
   ```bash
   # Windows
   set STOCK_DB=\\server\share\vfx\stax_production.db
   
   # Linux/macOS
   export STOCK_DB=/server/share/vfx/stax_production.db
   ```


---
</details>

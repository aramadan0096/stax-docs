# 3D Mesh Ingestion

This document describes how StaX converts, ingests, and previews 3D mesh assets (GLB/OBJ/FBX/Alembic/etc.). It explains the conversion pipeline, ingestion flow, the WebGL preview server/viewer, runtime configuration, troubleshooting steps, and developer notes so contributors can understand and extend the behavior.

![StaX graph](assets/images/3d_viewer.gif)

**Why this matters**
- 3D assets come in many formats with different exporters and quirks. StaX centralizes conversion into GLB for consistent WebGL previewing and downstream insertion/processing.

**Key files & locations**
- `src/glb_converter.py` — High-level conversion helpers and fallback logic.
- `dependencies/blender/convert_to_glb.py` — Production Blender-side conversion script (invoked via Blender CLI).
- `src/ingestion_core.py` — Ingestion pipeline and conversion invocation wrappers (streams Blender stdout to ingestion progress/notes).
- `src/geometry_viewer.py` — Local threaded HTTP server + Qt `GeometryViewerWidget` (uses QtWebEngine to load the WebGL viewer).
- `resources/geometry_viewer/index.html` — Web viewer that loads the GLB via an authenticated endpoint.
- `src/video_player_widget.py` — Integrates geometry viewer into the preview pane.
- `src/ui/media_display_widget.py` — Thumbnail generation (square 1:1 pipeline) and placeholder handling.
- `src/nuke_bridge.py` — Nuke insert/register functions (toolsets use `nuke.svg` placeholder).

Overview of the pipeline
------------------------
1. User triggers ingestion (single-file ingest or bulk library ingestion).
2. `IngestionCore.ingest_file(...)` executes. It decides the file type and copy policy.
3. For geometry files that are not already GLB, the ingestion path calls a conversion helper (via `_convert_geometry_asset` / `glb_converter.convert_to_glb`) which attempts:
   - Primary: Run Blender as a CLI process that executes the bundled script `dependencies/blender/convert_to_glb.py` to export a GLB.
   - Fallback: If the Blender export fails, attempt a Python-level fallback using libraries such as `trimesh` / `pygltflib` (when available) to read and re-export.
4. Blender conversion is run in streaming mode: the ingestion wrapper reads Blender stdout/stderr, writes progress notes to the ingestion progress mechanism, and enforces an idle timeout to avoid hangs.
5. The GLB output is written to the configured previews directory (used by the viewer). The ingestion then continues with thumbnail generation and database cataloging.
6. When a user selects the element in the UI, the `GeometryViewerServer` exposes a short-lived tokenized endpoint `/model/<token>` that streams the GLB; `GeometryViewerWidget` (Qt WebEngine) loads `resources/geometry_viewer/index.html?model=/model/<token>` to render it in WebGL.

Design details and developer notes
----------------------------------
- Blender-first approach: The project bundles a robust Blender conversion script (copied from the test suite) that iteratively strips unsupported export kwargs and retries exports until a supported combination succeeds. This gives the best fidelity for formats like FBX, OBJ, Alembic.
- Streaming output: `IngestionCore` contains a helper (`_run_blender_cli_conversion` or similar) that launches Blender and pumps its stdout into the UI ingestion progress notes in realtime. This provides user feedback while conversion runs.
- Idle timeout: Conversions include an idle timeout (configurable via `blender_idle_timeout`) to abort stale conversions that produce no output for a long time.
- Tokenized model endpoints: To avoid direct file paths and enable ephemeral access, the `GeometryViewerServer` serves a tokenized route that maps to a temporary GLB stream; tokens are short-lived and tied to the running process.
- Viewer assets: The Web viewer depends on the cgwire `js-3d-model-viewer` library (https://cgwire.github.io/js-3d-model-viewer/) — this library is bundled under `dependencies/` and is loaded by the small HTML wrapper (`resources/geometry_viewer/index.html`). The Qt `QWebEngineView` loads this page locally via the internal HTTP server.

Configuration
-------------
- `config.json` and the runtime `Config` object can include these keys (examples):
  - **`blender_path`**: Optional. Full path to Blender executable. If unset, the system `blender` on PATH is used.
  - **`blender_idle_timeout`**: Idle timeout in seconds for Blender conversions (defaults present in code; e.g. 900s).
  - **`previews_path`**: Directory where GLB preview files are written and served from.
  - **`nuke_mock_mode`**: If true, Nuke integration is mocked.

How to run a conversion manually (developer)
-------------------------------------------
- To test Blender conversion manually without running the UI:
  - Ensure Blender is in PATH or set `blender_path` to its executable.
  - Run Blender in background with the conversion script: (PowerShell)

```powershell
blender --background --python dependencies\blender\convert_to_glb.py -- "C:\path\to\asset.fbx" "C:\temp\out.glb"
```

- The script prints export attempts and warnings; it will write the GLB at the target path on success.

Viewer details
--------------
- The `GeometryViewerServer` exposes:
  - `GET /viewer/index.html` (static viewer client)
  - `GET /model/<token>` streams a GLB backing file mapped to the token
- `GeometryViewerWidget` wraps a `QWebEngineView`, loads the viewer HTML with the model token query param, and manages server lifecycle.
- The viewer uses the GLB middle-frame for sequences (if ingestion produced GLB from an image sequence) and ensures typical PBR support via the `js-3d-model-viewer` capabilities.

Thumbnailing and UI
-------------------
- StaX generates a fixed square 1:1 thumbnail pipeline for gallery consistency. Thumbnails are built via `_build_fixed_thumbnail(pixmap, size)` in `src/ui/media_display_widget.py`.
- Placeholders (e.g., `nuke.svg`, `cube.svg`) are passed through the same pipeline to ensure consistent layout and sizing.

Troubleshooting
---------------
- Blender conversion stalls or hangs:
  - Check `blender_path` configuration and ensure Blender is accessible.
  - Re-run conversion manually with the command shown above to capture Blender stdout/stderr.
  - Increase `blender_idle_timeout` in config if Blender legitimately takes long.
- No preview visible in UI (geometry not rendering):
  - Confirm GLB exists in `previews_path` for the ingested element.
  - Check application logs for `GeometryViewerServer` token generation and `/model/<token>` requests.
  - Ensure QtWebEngine is available in your Python environment (PySide2 with QtWebEngine).
- Preview appears as broken or empty:
  - Try opening the GLB in an external viewer (e.g., https://gltf-viewer.donmccurdy.com/) to validate the GLB itself.
  - If external viewer works, check console output of the embedded WebEngine (enable remote debugging or inspect logs) to find runtime WebGL errors.

Extending the pipeline
----------------------
- Add format-specific pre-processing hooks in `src/ingestion_core.py` to run before conversion (e.g., clean up FBX nodes). Use `ProcessorManager` hooks for pre/post-ingest processors.
- To add an additional fallback (e.g., Blender + FBX SDK), update `src/glb_converter.py` to include a new branch in `convert_to_glb()` that tries the SDK path before Python fallbacks.
- To change viewer features (e.g., add camera presets, lighting controls), edit `resources/geometry_viewer/index.html` and the bundled `js-3d-model-viewer` configuration.

Developer quick check list (manual tests)
-----------------------------------------
- Ingest a `.glb` file: should skip conversion and produce preview immediately.
- Ingest a `.fbx` file: should run Blender conversion, stream logs into the ingestion progress, and produce GLB preview when done.
- With a converted GLB in previews, select the element in the UI and confirm the `GeometryViewerWidget` loads and displays the model.
- If conversion fails, inspect ingestion notes/logs for Blender stdout and fallback attempts.

Log and diagnostics tips
------------------------
- Ingestion logs will include streaming Blender stdout lines and conversion attempt messages. Look for export attempt messages in `dependencies/blender/convert_to_glb.py` style logs.
- Server-side viewer logs: the HTTP server will log requests to `/model/<token>`; token mapping errors indicate either missing GLB or expired token.

Notes and caveats
-----------------
- The project aims for robust Blender-first conversion for fidelity, but this depends on a working Blender CLI in the runtime environment.
- Some Python libraries used for fallbacks (`trimesh`, `pygltflib`) may not be available in every environment; treat them as optional fallbacks.
- The ingestion pipeline is written to stream and report progress; tune `blender_idle_timeout` conservatively for large conversions.

Contact & further help
----------------------
- If you need changes to conversion heuristics or want to add support for more formats, open an issue or a PR and reference this document. For urgent debugging, run the manual Blender conversion command and attach Blender stdout/stderr logs.

----

File created: `docs/3D_Mesh_Ingestion.md` — please tell me if you'd like this added to `README.md`, linked from the app Help menu, or expanded with code snippets from the actual functions.

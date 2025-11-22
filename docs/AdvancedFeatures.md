
## Advanced Features

### Pagination

For large libraries (1000+ assets), enable pagination in Settings → Network & Performance:
- Set items per page (50, 100, 200, 500)
- Navigate with First/Previous/Next/Last buttons
- Improves performance and reduces memory usage

### Preview System

StaX generates three types of previews:
- **Static Thumbnail**: PNG image for quick display
- **Animated GIF**: 3-second loop for hover preview
- **Video Preview**: Low-res MP4 for playback in preview pane (standalone mode)
- **3D Scene Preview**: Embedded WebGL viewport for GLB/GLTF assets with orbit and zoom controls. StaX uses the bundled `js-3d-model-viewer` (a lightweight CGwire-derived Javascript 3D Model Viewer) for the in-app WebGL experience. When assets are provided in other formats (FBX, OBJ, Alembic, etc.), StaX will attempt to convert them to GLB — the preferred conversion path is a Blender CLI script (tracked as `src/convert_to_glb.py`), with fallbacks to Python libraries such as `trimesh` where available.

Hover over elements in gallery view to play GIF previews automatically.

### Network-Aware Database

For multi-user studios:
- SQLite database with file locking prevents conflicts
- Supports concurrent access from multiple workstations
- WAL (Write-Ahead Logging) journal mode for better concurrency
- Configurable retry logic and timeouts

### Tags and Metadata

- Add tags to elements for better searchability (comma-separated)
- Edit metadata: frame range, comment, tags, deprecated status
- Right-click → "Edit Metadata" to update element properties

---

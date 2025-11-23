
## Configuration

![StaX graph](assets/images/StaX-settings-standalone.png)

### Database Location

**Environment Variable (Recommended for Studios):**
```bash
# Set before launching Nuke or standalone app
export STOCK_DB=/path/to/shared/database.db
```

**Config File:**

Edit `config/config.json`:
```json
{
    "database_path": "./data/stax.db",
    "default_repository_path": "./repository",
    "preview_dir": "./previews",
    "default_copy_policy": "soft",
    "nuke_mock_mode": false
}
```
![StaX graph](assets/images/StaX-login-standalone.png)

### Settings Panel

Open with `Ctrl+3` to configure:

- **General**: Database location, user preferences
- **Ingestion**: Copy policy, sequence detection settings, Blender executable override for geometry conversion
- **Preview & Media**: Thumbnail size, GIF settings, video preview options
- **Network & Performance**: Database retries/timeout, cache settings, pagination
- **Custom Processors**: Pre-ingest, post-ingest, and post-import hook scripts
- **Security & Admin**: User management, password changes (admin only)

![StaX graph](assets/images/StaX-settings-standalone1.png) 

### Custom Processors (Pipeline Integration)

StaX supports custom Python scripts at three points in the workflow:

1. **Pre-Ingest Processor**: Runs before files are copied (validation, naming enforcement)
2. **Post-Ingest Processor**: Runs after cataloging (metadata injection, external system notifications)
3. **Post-Import Processor**: Runs after Nuke node creation (OCIO setup, expression application)

![StaX graph](assets/images/StaX-settings-standalone2.png) 

**Example Pre-Ingest Hook:**
```python
# processors/validate_naming.py
import re

# Context contains: {'name': str, 'filepath': str, 'element_data': dict}
if not re.match(r'^[A-Z]{3}_\d{4}', context['name']):
    result = {
        'continue': False,
        'message': 'File name must match pattern: XXX_####'
    }
else:
    result = {'continue': True}
```

Configure processor paths in Settings → Custom Processors tab.

---

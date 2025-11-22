
## Troubleshooting

### Nuke Plugin Not Loading

**Check Script Editor output:**
```python
import nuke
print(nuke.pluginPath())
# Should include StaX directory
```

**Verify init.py executed:**
- Look for `[StaX] Plugin loaded from:` message

**Common issues:**
- Missing `PySide2`: Install in Nuke's Python environment
- Wrong `NUKE_PATH`: Verify environment variable points to StaX folder
- Permission errors: Check read/write access to StaX directory

### Database Lock Errors

**Symptoms:**
- "database is locked" error messages

**Solutions:**
1. Close other StaX instances on the same machine
2. Check for stale `.lock` files in database directory
3. Increase retry timeout in Settings → Network & Performance

### Drag & Drop Not Working in Nuke

**Causes:**
1. Running in mock mode (check `nuke_mock_mode` setting)
2. Real Nuke API not detected
3. No elements selected

**Debug:**
```python
# In Nuke Script Editor
import nuke_launcher
print("Nuke mode:", nuke_launcher.NUKE_MODE)
```

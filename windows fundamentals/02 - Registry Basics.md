# Registry Basics

Think of the Registry as Windows' brain - it stores every setting, preference, and configuration for the OS and every installed app.

### Structure
It's like a folder tree:
- `HKEY_LOCAL_MACHINE (HKLM)` → Machine-wide settings (all users)
- `HKEY_CURRENT_USER (HKCU)` → Settings for the logged-in user only
- `HKEY_CLASSES_ROOT (HKCR)` → File type associations (.exe, .pdf etc)
- `HKEY_USERS (HKU)` → All user profiles
- `HKEY_CURRENT_CONFIG (HKCC)` → Hardware settings

You only really need to know HKLM and HKCU for security work.

### Why it matters for security
Malware loves the registry for persistence - making sure it runs every time Windows starts.

### The two most abused registry keys:
- `HKCU\Software\Microsoft\Windows\CurrentVersion\Run`
- `HKLM\Software\Microsoft\Windows\CurrentVersion\Run`

Anything listed here auto-runs on startup. Malware adds itself here to survive reboots.

## Open Registry Editor
1. Press `Win + R` → type `regedit` → Enter
2. Then navigate to:
   - `HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run`
   - You'll see programs set to auto-start for your user.
3. Or via PowerShell:
```powershell
# Check startup entries for current user
Get-ItemProperty "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run"
```
```powershell
# Check machine-wide startup entries
Get-ItemProperty "HKLM:\Software\Microsoft\Windows\CurrentVersion\Run"
```

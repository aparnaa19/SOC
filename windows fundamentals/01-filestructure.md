# Windows File System Basics

Think of Windows like a filing cabinet. Everything has its place.

### The Drive

Windows is usually installed on `C:\` (the **C drive**).  
This is where the operating system, installed programs, and user files are stored.

---

### Key Folders to Know

| Folder | What it contains |
|--------|-------------------|
| `C:\Windows\` | Core Windows operating system files |
| `C:\Windows\System32\` | Critical system files, DLLs, and executables |
| `C:\Program Files\` | Installed 64-bit applications |
| `C:\Program Files (x86)\` | Installed 32-bit applications |
| `C:\Users\` | One folder for each user account |
| `C:\Users\<username>\AppData\` | Hidden folder that stores application data and settings |
| `C:\Temp\` or `C:\Users\<username>\AppData\Local\Temp\` | Temporary files used by the system and applications |

---

### Why This Matters for Security

These folders are important from a security perspective because malware often hides or drops files in places that users rarely check.

Common targets include:

- `System32` because it is trusted and contains legitimate system files
- `AppData` because it is user-specific, hidden by default, and commonly writable
- `Temp` folders because they are frequently ignored and used for temporary execution

---

## User Accounts & Permissions

Windows has a simple hierarchy:

- **System**
  
  ↓
- **Administrator**
  
  ↓
- **Standard User**
  
  ↓
- **Guest**

| Account Type   | Can do                                                                                     |
|----------------|--------------------------------------------------------------------------------------------|
| **Administrator** | Install software, change system settings, access all files                            |
| **Standard User**   | Run apps, change own settings only                                                    |
| **Guest**           | Very limited, mostly read-only                                                        |
| **SYSTEM**          | Hidden - used by Windows itself. Most powerful account that exists                   |

Malware that escalates to SYSTEM owns the machine.

## UAC - User Account Control

That popup saying "Do you want to allow this app to make changes?" - that's UAC. It's a speed bump between standard and admin actions.


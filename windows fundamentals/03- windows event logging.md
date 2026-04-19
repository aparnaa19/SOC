# Windows Event Logging

Think of Windows Event Logs as your machine's diary — every important action gets written down. As a security analyst, this diary is your best friend during an investigation.

### What Are Audit Policies?
By default, Windows doesn't log everything. You have to tell it what to watch. That's what Audit Policies are — rules that say "hey Windows, write this down when it happens."

### How to enable them
- Open PowerShell as Admin:
```powershell
# See your current audit policy settings
auditpol /get /category:*
```
- You'll see something like:
  - Logon/Logoff → Success and Failure
  - Process Creation → No Auditing ← this means process creation isn't being logged!
- Enable Process Creation logging:
```powershell
auditpol /set /subcategory:"Process Creation" /success:enable /failure:enable
```

### Why it matters
Without this enabled, Event ID 4688 (process creation) won't be logged — meaning malware can run silently.

## Key Event IDs

Event IDs are just numbered codes Windows assigns to specific actions. Think of them like error codes but for security events.

### The 4 you must know:

| Event ID | What it means | Why it matters |
| --- | --- | --- |
| **4688** | A new process was created | Malware launching = new process |
| **4624** | Successful login | Who logged in and when |
| **4104** | PowerShell script executed | Malicious scripts leave traces here |
| **7045** | A new service was installed | Malware installs itself as a service |

# Event Viewer & Filtering Logs

Event Viewer is the GUI way to read Windows logs. Think of it as a search interface for your machine's diary.

### How to open it
- Win + R → type: `eventvwr.msc` → Enter
- Or via PowerShell:
```powershell
powershell -Command "start-process eventvwr.msc"
```

### The layout — 3 things to know:
```
Event Viewer
├── Windows Logs
│   ├── Application  → app crashes, errors
│   ├── Security     → logins, process creation ← YOU CARE ABOUT THIS
│   ├── Setup        → installation events
│   └── System       → service starts/stops, driver issues
└── Applications and Services Logs
    └── Microsoft → Windows → PowerShell ← 4104 lives here
```

### Security log = your main investigation target
### PowerShell log = where script execution lives

### Filter for a specific Event ID
1. Open Event Viewer.
2. Click **Windows Logs** → **Security**.
3. On the right panel, click **"Filter Current Log"**.
4. In the **Event ID** box, type `4624`.
5. Hit **OK**.

You'll now see only login events.

### Do the same via PowerShell:
```powershell
# Filter Security log for logins (4624)
Get-WinEvent -LogName Security -FilterXPath "*[System[EventID=4624]]" | Select-Object -First 5
```
```powershell
# Filter for PowerShell scripts (4104)
Get-WinEvent -LogName "Microsoft-Windows-PowerShell/Operational" -FilterXPath "*[System[EventID=4104]]" | Select-Object -First 5
```

## What a single event looks like:
When you click on any event, you'll see:
- **Event ID:** 4624
- **Date/Time:** 2026-04-19 10:23:01
- **Computer:** DESKTOP-ABC
- **Description:** An account was successfully logged on.
 
> **Account Name:** Aparnaa  
> **Logon Type:** 2 (interactive = physical login)  
> **IP Address:** -
 
### Logon Types to know:
type | meaning 
---|---
2 | Interactive (sat at the machine) 
3 | Network (remote access) 
10 | Remote Interactive (RDP)
 
Logon Type 10 from an unknown IP = red flag.


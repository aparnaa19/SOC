# Suspicious PowerShell Investigation - Cheat Sheet

---

## Red Flags in PowerShell Commands

| Flag | What it does | Why suspicious |
|---|---|---|
| `-EncodedCommand` / `-enc` / `-e` | Runs Base64 encoded command | Hides real intent |
| `-ExecutionPolicy Bypass` | Skips script restrictions | Legit scripts don't need this |
| `-WindowStyle Hidden` / `-w hidden` | Invisible window | Evasion |
| `-NonInteractive` / `-noni` | No user prompts | Stealth automation |
| `-NoProfile` / `-nop` | Skips security profile | Bypass restrictions |
| `-Version 2` | Downgrades PowerShell version | Bypasses script block logging entirely |
| `IEX` / `Invoke-Expression` | Executes a string as code | Fileless malware |
| `DownloadString` / `WebClient` | Downloads from internet | C2 communication |
| `FromBase64String` | Decodes Base64 | Hidden payload |
| `GzipStream` | Decompresses data | Double obfuscation |
| `VirtualAlloc` + `CreateThread` | Injects code into memory | Shellcode injection |
| `AmsiScanBuffer` / `amsiInitFailed` | Disables AMSI scanning | Security tool bypass |
| `[Reflection.Assembly]::Load` | Loads assembly from memory | Fileless execution |
| `GetDelegateForFunctionPointer` | Calls Windows API directly | Reflective injection |
| `sekurlsa` / `logonpasswords` / `mimikatz` | Credential dumping keywords | Credential theft |
| `$ExecutionContext.SessionState.LanguageMode` | Checks language mode | Constrained language bypass |

---

## Suspicious Parent Processes

Normal parents for PowerShell: `explorer.exe`, `cmd.exe`, `svchost.exe`

Suspicious parents: `winword.exe`, `excel.exe`, `outlook.exe`, `chrome.exe`, `firefox.exe`, `mshta.exe`, `wscript.exe`, `cscript.exe`, `certutil.exe`, `bitsadmin.exe`, `regsvr32.exe`

---

## Key Event IDs

### Security Log
| Event ID | What it tells you |
|---|---|
| 4688 | New process created - what ran and what launched it |
| 4657 | Registry value modified |
| 4698 | Scheduled task created |
| 4699 | Scheduled task deleted (attacker covering tracks) |
| 4702 | Scheduled task updated |
| 4697 | New service installed |
| 7045 | New service installed (System log) |

### PowerShell Operational Log
| Event ID | What it tells you |
|---|---|
| 4103 | Pipeline execution details and output |
| 4104 | Script block content - actual decoded script (most important) |
| 4105 | Script block logging started |
| 4106 | Script block logging stopped |

### Windows PowerShell Log
| Event ID | What it tells you |
|---|---|
| 400 | PowerShell engine started, shows host application |
| 800 | Pipeline execution details |

### Sysmon Log
| Event ID | What it tells you |
|---|---|
| 1 | Process creation with hashes (more detailed than 4688) |
| 3 | Network connection - C2 check |
| 7 | DLL loaded into process |
| 8 | CreateRemoteThread - cross process injection |
| 11 | File created on disk |
| 13 | Registry value set |
| 22 | DNS query made |

---

## PowerShell Logging Types

| Logging Type | What it captures | How to enable |
|---|---|---|
| Script Block Logging (4104) | Decoded script content | Group Policy - Turn on Script Block Logging |
| Module Logging (4103) | All pipeline executions | Group Policy - Turn on Module Logging |
| Transcription | Full session transcript saved to file | Group Policy - Turn on PowerShell Transcription |
| Process Creation (4688) | Command line in logs | Group Policy - Audit Process Creation + Include command line |

If any of these are not enabled on a machine, you have a visibility gap. Always document it in your report.

---

## Investigation Order

1. Event 4688 - did PowerShell run? What launched it? What was the command line?
2. Event 4104 - what did the script actually do?
3. Decode the payload if encoded
4. Event 3 / Event 22 - did it phone home? What domains or IPs did it reach?
5. Event 7 / Event 8 - did it inject into another process?
6. Event 11 - did it drop any files to disk?
7. Event 4698 / Event 13 / Event 7045 - did it persist?
8. Scope - did the same activity happen on other machines?

---

## KQL Queries for Elastic

### Detection Queries

```
# Find all script block logs
winlog.event_id : 4104

# Find encoded commands
winlog.event_id : 4104 AND powershell.file.script_block_text : ("*EncodedCommand*" OR "*Encoded Command*" OR " -enc " OR " -e ")

# Find download cradles
winlog.event_id : 4104 AND powershell.file.script_block_text : (*DownloadString* OR *WebClient* OR *IEX*)

# Find execution policy bypass
winlog.event_id : 4104 AND powershell.file.script_block_text : "*ExecutionPolicy*Bypass*"

# Find hidden window
winlog.event_id : 4104 AND powershell.file.script_block_text : "*WindowStyle*Hidden*"

# Find AMSI bypass attempts
winlog.event_id : 4104 AND powershell.file.script_block_text : (*AmsiScanBuffer* OR *amsiInitFailed* OR *AmsiUtils*)

# Find reflective loading
winlog.event_id : 4104 AND powershell.file.script_block_text : (*Reflection.Assembly* OR *GetDelegateForFunctionPointer*)

# Find credential dumping keywords
winlog.event_id : 4104 AND powershell.file.script_block_text : (*mimikatz* OR *sekurlsa* OR *logonpasswords* OR *lsadump*)

# Find suspicious parent process
winlog.event_id : 4688 AND winlog.event_data.ParentProcessName : (*winword* OR *excel* OR *outlook* OR *chrome* OR *mshta* OR *wscript* OR *cscript*)

# Find PowerShell process creation
winlog.event_id : 4688 AND process.name : "powershell.exe"

# Find PowerShell downgrade attack
winlog.event_id : 4688 AND process.command_line : "*-version 2*"

# Find renamed PowerShell (evasion technique)
winlog.event_id : 4688 AND process.executable : "*WindowsPowerShell*" AND NOT process.name : "powershell.exe"
```

### Investigation Queries (replace MACHINE_NAME with actual hostname)

```
# All processes on a machine
winlog.computer_name : "MACHINE_NAME" AND winlog.event_id : 4688

# Network connections from a machine
winlog.computer_name : "MACHINE_NAME" AND winlog.event_id : 3

# DNS queries from a machine
winlog.computer_name : "MACHINE_NAME" AND winlog.event_id : 22

# DLL loads on a machine
winlog.computer_name : "MACHINE_NAME" AND winlog.event_id : 7

# Cross process injection on a machine
winlog.computer_name : "MACHINE_NAME" AND winlog.event_id : 8

# Files dropped to disk on a machine
winlog.computer_name : "MACHINE_NAME" AND winlog.event_id : 11

# Scheduled task created on a machine
winlog.computer_name : "MACHINE_NAME" AND winlog.event_id : 4698

# Registry modification on a machine
winlog.computer_name : "MACHINE_NAME" AND winlog.event_id : 13

# Script block logs on a machine
winlog.computer_name : "MACHINE_NAME" AND winlog.event_id : 4104
```

---

## How to Decode Payloads

**Base64 only (standard):**
```python
import base64
decoded = base64.b64decode("PASTE_HERE").decode("utf-8")
print(decoded)
```

**Base64 + UTF-16LE (PowerShell -EncodedCommand):**
```python
import base64
decoded = base64.b64decode("PASTE_HERE").decode("utf-16-le")
print(decoded)
```

**Base64 + Gzip (compressed payload):**
```python
import base64, gzip
decoded = gzip.decompress(base64.b64decode("PASTE_HERE"))
print(decoded.decode("utf-8"))
```

**CyberChef alternative:** gchq.github.io/CyberChef

| Encoding type | CyberChef recipe |
|---|---|
| Base64 only | From Base64 |
| PowerShell -EncodedCommand | From Base64, then Decode text UTF-16LE |
| Base64 + Gzip | From Base64, then Gunzip |

---

## Important Fields in Elastic

| What you want to know | Field name |
|---|---|
| What process ran | `process.name` |
| Full command used | `process.command_line` |
| What launched it | `process.parent.name` |
| Full path of parent | `process.parent.executable` |
| Which machine | `winlog.computer_name` |
| Which user | `winlog.event_data.SubjectUserName` |
| Script content | `powershell.file.script_block_text` |
| When | `@timestamp` |
| Destination IP | `destination.ip` |
| Destination port | `destination.port` |
| Source IP | `source.ip` |
| DNS query made | `dns.question.name` |
| File created path | `file.path` |
| DLL loaded | `winlog.event_data.ImageLoaded` |
| Process hash | `process.hash.sha256` |
| Token elevation type | `winlog.event_data.TokenElevationType` |

---

## Severity Decision

| Finding | Severity |
|---|---|
| Encoded command only | Low - investigate further |
| Download cradle detected | Medium - escalate |
| Suspicious parent process | High - isolate |
| AMSI bypass attempt | High - isolate |
| Shellcode injection in memory | Critical - incident response |
| Credential dumping keywords found | Critical - incident response |
| Persistence found | Critical - incident response |

---

## MITRE ATT&CK Mapping

| ID | Name | When you see it |
|---|---|---|
| T1059.001 | PowerShell | Any suspicious PowerShell execution |
| T1027 | Obfuscated Files or Information | Base64, Gzip, XOR encoding |
| T1055 | Process Injection | VirtualAlloc + CreateThread in script |
| T1106 | Native API | Direct Windows API calls via reflection |
| T1140 | Deobfuscate/Decode Files | Attacker decodes payload at runtime |
| T1218 | Signed Binary Proxy Execution | mshta, regsvr32, certutil launching PowerShell |
| T1003 | OS Credential Dumping | Mimikatz keywords in script content |
| T1562.001 | Disable Security Tools | AMSI bypass attempts |
| T1569.002 | Service Execution | Attack starts from services.exe |
| T1053.005 | Scheduled Task | New task created for persistence |
| T1547.001 | Registry Run Keys | Run key modified for persistence |
| T1070 | Indicator Removal | Clearing event logs, deleting scheduled tasks |
| T1134 | Access Token Manipulation | Token elevation type changes in 4688 |

---

## No Results - What to Check

If your 5 detection queries return no results:

1. Run `winlog.event_id : 4104` with no filters - do you get any results at all?
2. If yes - script block logging is working but no suspicious activity matched
3. If no - script block logging may not be enabled, document as a visibility gap
4. Run `winlog.event_id : 4688 AND process.executable : "*powershell*"` to check if PowerShell even ran
5. If PowerShell ran but no 4104 events - attacker may have used `-Version 2` downgrade to bypass logging


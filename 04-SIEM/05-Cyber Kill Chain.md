# Cyber Kill Chain

- The Cyber Kill Chain was created by Lockheed Martin to map the exact steps an attacker takes from zero to full compromise.
- It breaks every attack into 7 phases.
- As a SOC analyst, the job is to detect and break this chain at any phase — stop one link and the entire attack fails.
- To understand how this works in a real attack, I mapped each phase to the Emotet banking trojan campaign documented by CISA.

---

## Cyber Kill Chain has 7 phases:

### Phase 1 — Reconnaissance

- The attacker opens his laptop and starts watching.
- He searches LinkedIn to identify employees and the technologies the company uses.
- He runs Shodan to find exposed ports and services.
- He uses Maltego to map email addresses, colleagues, and vendors.
- He runs Nmap to fingerprint open ports and running services.
- He hasn't touched the target's systems yet.
- He's collecting OSINT. Zero alerts fired. Zero footprint.
- He now knows exactly what door to walk through.

### Phase 2 — Weaponization

- The attacker goes offline.
- The target has no idea what's coming.
- He writes a macro-embedded Word document — looks like a legitimate invoice.
- Inside it he hides a Meterpreter payload — a reverse shell that calls back to his C2 server the moment it executes.
- He crafts a phishing email template spoofing a trusted vendor.
- No network contact.
- No alerts.
- The weapon is built entirely in his own lab.
- This phase is invisible to defenders.

### Phase 3 — Delivery

- The attacker hits send.
- An employee receives an email — "Invoice_Final.docx" — from what looks like a trusted vendor.
- The email passed SPF but the attacker used a lookalike domain.
- It lands in the inbox, not spam.
- This is the transfer of the weapon from the attacker's environment into the target's environment.
- The malware is now one click away from detonation.
- The delivery vector: spear phishing email.

### Phase 4 — Exploitation

- The employee opens the document.
- Word says "Enable Content." They click it.
- The macro executes.
- It spawns a cmd.exe child process under winword.exe and runs a PowerShell cradle that downloads the second-stage payload from the attacker's staging server.
- The vulnerability just fired. The code just ran on the target machine. This is the moment of breach. The employee sees nothing. Word looks completely normal.
- The attacker is in.

### Phase 5 — Installation

- The attacker doesn't want to depend on that employee clicking again.
- The payload drops a RAT into a hidden directory under AppData.
- It writes a registry run key so it survives reboot.
- It adds a scheduled task that relaunches it every 15 minutes if killed.
- It drops a web shell on the exposed web server found during recon.
- The attacker now has persistence.
- He can come back tomorrow, next week, next month. He owns that machine quietly and permanently.

### Phase 6 — Command & Control (C2)

- The RAT phones home.
- Every 60 seconds it sends a beacon — an outbound HTTPS request — to a domain the attacker registered recently.
- It tunnels through port 443 to blend in with normal web traffic.
- The attacker receives the beacon on his Cobalt Strike C2 server and gets a live session.
- He can now run commands, move files, take screenshots — all through that encrypted tunnel.
- He has a remote keyboard inside the target network.
- The network team sees HTTPS traffic. Nothing looks unusual.

### Phase 7 — Actions on Objectives

- The attacker's goal: exfiltrate the database.
- He runs BloodHound to map Active Directory and find a path to a Domain Admin account.
- He dumps credentials using Mimikatz.
- He laterally moves to the database server using Pass-the-Hash.
- He compresses the data, splits it into small chunks, and slowly exfiltrates it over the C2 channel across several days — slow enough to avoid bandwidth alerts.
- By the time the SOC team gets an alert — if they ever do — the data is already gone.

---

## Considering Emotet: https://www.cisa.gov/news-events/alerts/2018/07/20/emotet-malware

### Phase 1 — Reconnaissance

Not documented in the CISA report.

> MITRE: T1595 — Active Scanning, T1589 — Gather Victim Identity Information

### Phase 2 — Weaponization

Not documented in the CISA report.

> MITRE: T1587 — Develop Capabilities

### Phase 3 — Delivery

- Emotet was delivered through malspam emails using familiar branding - imitating PayPal receipts, shipping notifications, and past-due invoices, even spoofing the MS-ISAC name to appear legitimate.
- Attachments included malicious links, PDFs, and macro-enabled Word documents.
- Emotet also used an Outlook scraper module to scrape names and email addresses from already compromised accounts and send additional phishing emails from those trusted accounts — restarting the delivery chain for new victims.

> MITRE: T1566 — Phishing, T1566.001 — Spearphishing Attachment

### Phase 4 — Exploitation

- When the victim clicked the malicious link, opened the PDF, or enabled the macro in the Word document included in the malspam, the exploit executed and Emotet ran on the victim machine for the first time.

> MITRE: T1059 — Command and Scripting Interpreter, T1204 — User Execution

### Phase 5 — Installation

- Emotet used auto-start registry keys and Windows services to persist across reboots.
- It used modular DLLs to continuously update its capabilities remotely without reinfecting the victim.
- It injected code into explorer.exe and other legitimate running processes to hide its presence.
- It created randomly-named files in system root directories running as Windows services.
- Once a new system was found via SMB, it wrote itself onto that disk - restarting the installation phase on the new machine, capable of infecting entire domains.

> MITRE: T1547 — Registry Run Keys, T1053 — Scheduled Task, T1055 — Process Injection

### Phase 6 — Command and Control

- Emotet connected to a remote C2 server through a generated 16-letter domain name ending in .eu making static domain blacklisting difficult.
- Once connected it reported the new infection, received configuration data, downloaded additional files, and awaited further instructions from the attacker.

> MITRE: T1071 — Application Layer Protocol, T1568 — Dynamic Resolution

### Phase 7 — Actions on Objectives

- Emotet collected sensitive system information including system name, location, and OS version and uploaded it to the C2 server.
- It then propagated across the network using five spreader modules:
  - NetPass.exe to recover stored network passwords
  - WebBrowserPassView to steal browser passwords
  - Mail PassView to steal email client passwords
  - Outlook Scraper to send phishing emails from compromised accounts to new victims
  - Credential Enumerator to brute force network accounts and spread Emotet to new machines via SMB shares - capable of infecting entire domains.

> MITRE: T1078 — Valid Accounts, T1110 — Brute Force, T1041 — Exfiltration Over C2

---

## Evasion Technique

- Emotet is Virtual Machine aware.
- It detects sandbox environments used by security analysts and generates false indicators to appear harmless.
- This is not limited to one Kill Chain phase.
- It operates throughout the entire attack lifecycle to avoid detection and analysis.

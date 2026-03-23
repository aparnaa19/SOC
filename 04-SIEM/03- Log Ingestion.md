# Log Ingestion

## Overview
Every system produces logs, but the **type, location, and format** of those logs can be different depending on:
- the operating system
- the application
- the device type
- the role of the system

### Examples
- **Windows** → Event Viewer logs
- **Linux** → log files in `/var/log/`
- **Web servers** → access logs and error logs
- **Firewalls** → allowed/blocked traffic logs
- **Authentication systems** → login and login-failure logs

### Conclusion
All systems generate logs, but not all systems generate the same kind of logs.

---

## A SIEM can analyse logs only when:
- the logs are being collected
- the logs are forwarded to the SIEM
- the SIEM supports that log format
- the logs are parsed and normalized properly

### SIEM can commonly analyze
- Windows event logs
- Linux logs
- firewall logs
- web server logs
- authentication logs
- IDS/IPS alerts

### SIEM may not analyze data properly when
- the system is not sending logs
- the format is unsupported
- the logs are incomplete or corrupted
- the data is not useful for security monitoring

---

## SIEM tools
There are many SIEM tools, and each one has different features, integrations, dashboards, and ways of ingesting logs.

### Common SIEM tools
- Splunk
- ELK / Elastic
- IBM QRadar
- ArcSight
- Microsoft Sentinel
- LogRhythm

### Differences between SIEM tools
- cost
- scalability
- cloud or on-premises support
- built-in detection rules
- supported integrations
- user interface and query language

---

## How to redirect logs into SIEM?
Many systems do not send logs directly on their own. Special tools or protocols are often used to collect and forward logs into a SIEM.

### Common log forwarding methods
- **Agent / Forwarder**  
  A lightweight software installed on the endpoint to collect and send logs to the SIEM.

- **Syslog**  
  A common protocol used to send logs from Linux systems, firewalls, routers, switches, and other devices.

- **Manual Upload**  
  Some SIEM tools allow offline log files to be uploaded for analysis.

- **Port-based forwarding**  
  Devices can be configured to send logs to a SIEM listening on a specific port.

### Examples of log shippers / forwarders
- Splunk Forwarder
- Winlogbeat
- Filebeat
- Logstash
- Fluentd
- Syslog

---

## Common Log Sources

### Windows Machine
Windows stores logs in **Event Viewer**.  
Each event is assigned a unique **Event ID**, which helps analysts identify the activity.

Examples of Windows logs:
- user logon events
- failed login attempts
- application errors
- system events
- security events

---

### Linux Machine
Linux stores logs in files, usually under `/var/log/`.

### Common Linux log locations
- `/var/log/httpd` → HTTP request/response and error logs
- `/var/log/cron` → cron job activity
- `/var/log/auth.log` or `/var/log/secure` → authentication logs
- `/var/log/kern` → kernel-related events

Examples of Linux logs:
- SSH login attempts
- cron jobs
- service errors
- authentication failures
- kernel messages

---

### Web Server
Web servers, such as Apache, generate logs for all incoming requests and server responses.

### Apache logs can show
- source IP address
- date and time
- HTTP method
- requested URL/path
- response code
- browser or user agent

These logs are useful for identifying:
- suspicious requests
- scanning activity
- exploitation attempts
- unusual traffic patterns

---
## Conclusion 
### What is Log Ingestion?
**Log ingestion** is the process of collecting logs from different systems and sending them into the SIEM for monitoring and analysis.

### Why it matters
Without log ingestion, the SIEM cannot see what is happening across the environment.

### Common ingestion methods
- Agent / Forwarder
- Syslog
- Manual Upload
- Port Forwarding

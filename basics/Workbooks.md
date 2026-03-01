# SOC Workbooks 
-  also called as playbooks/ runbooks/ investigation workflows
-  provide structured guidance for analysts when handling alerts and security incidents
-  aligned with organizational response standards
-  help analysts reduce response time by outlining predefined investigation steps and recommended actions

---
Before getting to know more about workbooks we will go through certain terms to get an idea:
## 1. Threat Intelligence Services
- provide context about indicators such as IP addresses, domains, URLs, and file hashes.

SOC analysts use threat intelligence platforms to:

- Enrich alerts with reputation data
- Identify known malicious infrastructure
- Track threat actor campaigns
- Support investigation decision making

Common examples include:

- VirusTotal
- AbuseIPDB
- AlienVault OTX
- Recorded Future

---

## 2. Anonymization Detection Services
- help analysts identify users or systems attempting to conceal their identity or origin during network communication.

These services assist investigations involving:

- VPN usage
- Proxy networks
- Tor exit nodes
- Cloud hosting infrastructure

Examples include:

- Spur.us
- IPinfo
- Lumu Intelligence

Analysts use these tools to determine whether suspicious traffic originates from anonymized infrastructure commonly associated with malicious activity.

---

## 3. Identity Inventory

- has data or information about organizational users, accounts, and access privileges

SOC analysts reference identity information to:

- Validate user login activity
- Detect compromised accounts
- Investigate privilege escalation attempts
- Identify inactive or unauthorized users

Common identity sources include:

- Active Directory
- Azure AD / Entra ID
- IAM Platforms

---

## Splunk (SIEM Platform)

Splunk is a Security Information and Event Management (SIEM) platform used to collect, search, analyze, and visualize machine generated data from multiple sources.

By ingesting logs, metrics, and event telemetry, Splunk enables analysts to:

- Monitor security alerts
- Perform log correlation
- Investigate endpoint behavior
- Detect abnormal activity patterns

Common SOC use cases include:

- Failed authentication monitoring
- Suspicious process execution analysis
- Network anomaly detection
- Incident timeline reconstruction


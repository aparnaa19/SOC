## 4. Ports
 
A **port** is a number that identifies a specific service or application on a device. Think of an IP address as a building's street address, and the port as the apartment number inside.
 
- IP address gets you to the right machine.
- Port number gets you to the right service on that machine.
### Common Ports to Know
 
| Port | Protocol | Service |
|---|---|---|
| 20, 21 | FTP | File transfer |
| 22 | SSH | Secure remote access |
| 25 | SMTP | Email sending |
| 53 | DNS | Domain name resolution |
| 80 | HTTP | Web traffic |
| 443 | HTTPS | Encrypted web traffic |
| 3306 | MySQL | Database |
| 3389 | RDP | Windows remote desktop |
| 8080 | HTTP (alt) | Dev/proxy web servers |
 
### Port Ranges
 
| Range | Category |
|---|---|
| 0 – 1023 | Well-known ports (reserved for standard services) |
| 1024 – 49151 | Registered ports (used by applications) |
| 49152 – 65535 | Dynamic/ephemeral ports (used temporarily by clients) |
 

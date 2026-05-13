
## OSI Model - 7 Layers
 
The OSI (Open Systems Interconnection) model breaks network communication into 7 distinct layers. Each layer has a specific job and only communicates with the layers directly above and below it.
 
Data travels **down** the layers when sending, and **up** the layers when receiving.
 
**Memory aid:** *All People Seem To Need Data Processing* (Layer 7 → Layer 1)
 
### Layer Reference Table
 
| Layer | Name | What it Does | Examples |
|---|---|---|---|
| 7 | Application | Interface for user-facing network services | HTTP, FTP, SSH, DNS |
| 6 | Presentation | Data translation, encryption, compression | SSL/TLS, JPEG, ASCII |
| 5 | Session | Opens, maintains, and closes communication sessions | NetBIOS, RPC |
| 4 | Transport | Reliable delivery, segmentation, error checking | TCP, UDP |
| 3 | Network | Logical addressing, routing across networks | IP, ICMP, routers |
| 2 | Data Link | Communication within the same network, MAC addressing | Ethernet, switches |
| 1 | Physical | Raw bit transmission over hardware | Cables, Wi-Fi, fiber |
 
### Why This Matters
 
The OSI model makes troubleshooting systematic:
 
- Can't ping a device? → Layer 3 (Network/IP) issue.
- Cable unplugged? → Layer 1 (Physical) issue.
- SSL certificate error? → Layer 6 (Presentation) issue.
- Website loads but session keeps dropping? → Layer 5 (Session) issue.

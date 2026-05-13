### Router
 
A router connects **different networks** together and routes packets between them. It operates at **Layer 3 (Network)**.
 
- Your home router connects your local network (192.168.x.x) to the internet.
- It uses IP addresses to decide where to forward each packet.
- Contains a **routing table** — a map of known networks and the best path to reach them.
```
Internet → Router → Your devices
```
 
### Switch
 
A switch connects **devices within the same network**. It operates at **Layer 2 (Data Link)**.
 
- Uses MAC addresses to forward traffic only to the intended device.
- More efficient than a hub (which broadcasts to every device).
- Common in offices: all computers plug into a switch.
```
Computer A → Switch → Computer B  (same network)
```
 
### Firewall
 
A firewall monitors and filters incoming and outgoing network traffic based on rules. It can operate at multiple layers.
 
- **Blocks or allows** traffic based on IP address, port, protocol, or content.
- Sits between your network and the outside world.
- Can be hardware (a physical device) or software (Windows Firewall, iptables).
#### Types of Firewalls
 
| Type | How it Works |
|---|---|
| Packet filter | Checks IP/port headers only (fast, basic) |
| Stateful | Tracks connection state, smarter filtering |
| Application layer (WAF) | Inspects actual content, catches app-level attacks |
 
### Device Summary
 
| Device | Layer | Uses | Connects |
|---|---|---|---|
| Switch | Layer 2 | MAC addresses | Devices within a network |
| Router | Layer 3 | IP addresses | Different networks |
| Firewall | Layer 3–7 | Rules/policies | Network to internet (as a gatekeeper) |
 

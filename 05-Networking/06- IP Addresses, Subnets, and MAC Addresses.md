### IP Address
 
A unique numerical label assigned to every device on a network.
 
**IPv4** — 32-bit address written as four octets:
```
192.168.1.105
```
 
**IPv6** — 128-bit address, used as IPv4 addresses run out:
```
2001:0db8:85a3:0000:0000:8a2e:0370:7334
```
 
#### Private vs Public IP
 
| Type | Range | Used For |
|---|---|---|
| Private | 192.168.x.x, 10.x.x.x, 172.16–31.x.x | Internal networks (not routable on internet) |
| Public | Everything else | Devices directly on the internet |
 
Your home router has one public IP (facing the internet) and assigns private IPs to your devices internally.
 
### Subnet
 
A subnet divides a larger network into smaller segments. This improves performance and security by limiting how many devices can communicate directly.
 
A **subnet mask** defines which part of the IP address is the network and which part is the host:
 
```
IP Address:   192.168.1.105
Subnet Mask:  255.255.255.0  (or /24 in CIDR notation)
 
Network part: 192.168.1
Host part:    105
```
 
Devices on the same subnet can communicate directly. Devices on different subnets must go through a router.
 
**CIDR notation** is shorthand for the subnet mask:
 
| CIDR | Subnet Mask | Available Hosts |
|---|---|---|
| /24 | 255.255.255.0 | 254 |
| /16 | 255.255.0.0 | 65,534 |
| /8 | 255.0.0.0 | 16,777,214 |
 
### MAC Address
 
A **Media Access Control (MAC) address** is a hardware identifier burned into every network interface card (NIC) at the factory.
 
```
Example: 00:1A:2B:3C:4D:5E
```
 
- 48 bits, written as 6 pairs of hexadecimal digits
- Operates at **Layer 2 (Data Link)**
- Used for communication **within** a local network
- Unlike IP addresses, MAC addresses do not change (though they can be spoofed)
#### IP vs MAC — Key Difference
 
| | IP Address | MAC Address |
|---|---|---|
| Layer | Network (Layer 3) | Data Link (Layer 2) |
| Scope | Cross-network routing | Local network only |
| Assigned by | Network/DHCP | Manufacturer |
| Changes? | Yes (dynamic/static) | No (hardware-level) |
 

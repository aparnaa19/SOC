## Key Protocols
 
### DNS — Domain Name System
 
Translates human-readable domain names into IP addresses.
 
```
You type: google.com
DNS resolves: 142.250.80.46
Your browser connects to that IP
```
 
Without DNS, you would need to remember IP addresses for every website. DNS is the internet's phonebook.
 
### HTTP — HyperText Transfer Protocol
 
The protocol used to transfer web pages. Communication is **unencrypted** — anyone intercepting the traffic can read it.
 
- Operates on **Port 80**
- Request/response model: browser sends a request, server sends back a response
### HTTPS — HTTP Secure
 
HTTP with encryption layered on top via **TLS (Transport Layer Security)**. Traffic is encrypted end-to-end.
 
- Operates on **Port 443**
- Always prefer HTTPS over HTTP — it protects data in transit
### FTP — File Transfer Protocol
 
Used to transfer files between a client and a server. Sends data in **plaintext** (insecure).
 
- Operates on **Port 21** (control) and **Port 20** (data)
- SFTP (SSH File Transfer Protocol) is the secure alternative
### SSH — Secure Shell
 
Used to remotely access and control another machine securely over an encrypted connection.
 
- Operates on **Port 22**
- Common use: logging into a remote Linux server from your terminal
```bash
ssh username@192.168.1.10
```
 
### Protocol Summary
 
| Protocol | Port | Encrypted | Use |
|---|---|---|---|
| HTTP | 80 | No | Web browsing |
| HTTPS | 443 | Yes | Secure web browsing |
| FTP | 21 | No | File transfer |
| SSH | 22 | Yes | Remote access |
| DNS | 53 | No (by default) | Name resolution |
 
---

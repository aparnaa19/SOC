## TCP/IP — How Packets Travel
 
TCP/IP is the foundational communication language of the internet. It is two protocols working together:
 
- **IP (Internet Protocol)** — handles addressing and routing. Decides *where* data goes.
- **TCP (Transmission Control Protocol)** — handles reliable delivery. Ensures data *arrives correctly*.
### What is a Packet?
 
When you send data, it is not transmitted as one large chunk. It is broken into small pieces called **packets**. Each packet contains:
 
| Field | Purpose |
|---|---|
| Source IP | Where the packet came from |
| Destination IP | Where the packet is going |
| Sequence number | Position in the overall data stream, for reassembly |
| Payload | The actual chunk of data |
 
### How Packets Travel
 
```
Your Computer → Router → ISP Network → Destination Server
```
 
1. Data is broken into packets at the source.
2. Each packet is independently routed through the network — they may take different paths.
3. The destination receives all packets and reassembles them in order using sequence numbers.
4. If a packet is lost, TCP requests a re-send.
### TCP vs IP — Key Difference
 
| Protocol | Responsibility |
|---|---|
| IP | Address and route each packet |
| TCP | Track packets, confirm delivery, request re-sends if lost |

# Wireshark Basics and Packet Capture Challenges

## Goal

Learn to read packet captures in Wireshark and use filters and statistics to answer questions about the traffic in two PCAP challenges. This write-up focuses on the methods and commands I used.

## Wireshark skills covered

- Main window: menu bar, display filter bar, and the three panes: Packet List, Packet Details, Packet Bytes (hex and ASCII)
- Capture filters vs display filters (`tcp port 80` vs `tcp.port == 80`)
- Applying display filters to isolate traffic
- Following TCP streams to rebuild a conversation
- Right-click a field, then Apply as Column
- Statistics menu: Protocol Hierarchy, Conversations, Endpoints

---

## Exercise 1: Network Protocol Analysis
Practised identifying protocols, analysing ICMP and DNS traffic, and using endpoint statistics to investigate packet captures.

| Purpose | Wireshark filter / method |
|---|---|
| Identify the protocol using a specific port | `tcp.port == <port> \|\| udp.port == <port>` |
| Identify repeated ICMP echo requests | `icmp`, then inspect the destination addresses of echo request packets |
| Display DNS response packets | `dns.flags.response == 1`, then check the displayed packet count |
| Identify the host transmitting the most data | Statistics > Endpoints > IPv4, then sort by Tx Bytes |
---

## Exercise 2: HTTP and FTP Traffic Analysis

Practised inspecting application-layer traffic, identifying server banners, following TCP streams, and searching packet contents for relevant information.

| Purpose | Wireshark filter / method |
|---|---|
| Inspect HTTP POST requests and examine transmitted form data | `http`, then inspect POST requests and use Follow > HTTP Stream |
| Identify FTP server banner information | `ftp`, then inspect the server response banner |
| Inspect FTP file transfer activity | `ftp`, then follow the relevant TCP stream |
| Search packet contents for file references and examine timestamps | Edit > Find Packet, search for a relevant text string, then inspect the packet timestamp |

---

## Conclusions

- Sorting Endpoints by bytes is a fast way to find the busiest host.
- HTTP and FTP send data in plain text, so credentials and server versions can be read straight from the capture.
- Following a stream is much clearer than reading packets one by one.
- Searching packet contents by string saves a lot of time on large captures.
